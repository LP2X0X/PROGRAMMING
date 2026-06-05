---
tags:
 - csharp
 - ado-net
 - transactions
---

## Transactions -- Grouping Database Operations into Atomic Units

A **database transaction** is a sequence of operations that are treated as a single, indivisible unit of work. Either **all** operations in the transaction succeed and are permanently applied (**commit**), or **all** are undone as if they never happened (**rollback**). Transactions are essential for maintaining data integrity when multiple related changes must succeed or fail together.

---

## ACID Properties

Every database transaction guarantees the **ACID** properties:

| Property | Meaning | Example |
|---|---|---|
| **Atomicity** | All operations succeed or all are rolled back -- no partial state | A bank transfer debits one account and credits another; if the credit fails, the debit is reversed |
| **Consistency** | The database moves from one valid state to another; constraints are never violated | Foreign keys, unique constraints, and check constraints are enforced at commit |
| **Isolation** | Concurrent transactions do not see each other's uncommitted changes (degree depends on isolation level) | Two users updating the same row don't see each other's intermediate state |
| **Durability** | Once committed, the changes survive system crashes, power failures, etc. | Committed data is written to the transaction log on disk before acknowledgment |

---

## Basic Transaction Pattern with DbTransaction

The standard pattern in ADO.NET is: open connection, begin transaction, execute commands, commit or rollback.

```csharp
using var conn = new SqlConnection(connStr);
conn.Open();
using var tx = conn.BeginTransaction();

try
{
    using var cmd1 = new SqlCommand(
        "UPDATE Accounts SET Balance = Balance - 100 WHERE Id = @FromId", conn, tx);
    cmd1.Parameters.Add("@FromId", SqlDbType.Int).Value = 1;
    cmd1.ExecuteNonQuery();

    using var cmd2 = new SqlCommand(
        "UPDATE Accounts SET Balance = Balance + 100 WHERE Id = @ToId", conn, tx);
    cmd2.Parameters.Add("@ToId", SqlDbType.Int).Value = 2;
    cmd2.ExecuteNonQuery();

    tx.Commit();   // both updates are permanently applied
}
catch
{
    tx.Rollback(); // both updates are undone
    throw;
}
```

```ad-important
title: Every Command Must Be Associated with the Transaction
Every `DbCommand` executed within a transaction ==must have its `Transaction` property set== to the active transaction object. You can set it either:
1. Via the `SqlCommand` constructor: `new SqlCommand(sql, conn, tx)`
2. Via the property: `cmd.Transaction = tx;`

Forgetting this throws: `"Execute requires the command to have a transaction when the connection assigned to the command is in a pending local transaction."`
```

---

## Async Transaction Pattern

In async server applications, use the async counterparts:

```csharp
await using var conn = new SqlConnection(connStr);
await conn.OpenAsync();
await using var tx = await conn.BeginTransactionAsync();

try
{
    await using var cmd1 = new SqlCommand(
        "UPDATE Accounts SET Balance = Balance - @Amount WHERE Id = @FromId", conn, tx);
    cmd1.Parameters.Add("@Amount", SqlDbType.Decimal).Value = 100m;
    cmd1.Parameters.Add("@FromId", SqlDbType.Int).Value = 1;
    await cmd1.ExecuteNonQueryAsync();

    await using var cmd2 = new SqlCommand(
        "UPDATE Accounts SET Balance = Balance + @Amount WHERE Id = @ToId", conn, tx);
    cmd2.Parameters.Add("@Amount", SqlDbType.Decimal).Value = 100m;
    cmd2.Parameters.Add("@ToId", SqlDbType.Int).Value = 2;
    await cmd2.ExecuteNonQueryAsync();

    await tx.CommitAsync();
}
catch
{
    await tx.RollbackAsync();
    throw;
}
```

---

## Isolation Levels

The **isolation level** controls how much one transaction can see of another concurrent transaction's uncommitted changes. Higher isolation prevents more anomalies but reduces concurrency (more blocking/deadlocks).

### Concurrency Anomalies

| Anomaly | Description |
|---|---|
| **Dirty read** | Transaction A reads a row that Transaction B has modified but not yet committed. If B rolls back, A has read data that never existed. |
| **Non-repeatable read** | Transaction A reads a row, Transaction B modifies and commits it, then A reads the same row again and gets a different value. |
| **Phantom read** | Transaction A runs a query, Transaction B inserts new rows that match A's query criteria and commits, then A runs the same query and gets additional rows that weren't there before. |

### Isolation Level Matrix

| Isolation Level | Dirty Reads | Non-Repeatable Reads | Phantom Reads | Locking Behavior |
|---|---|---|---|---|
| `ReadUncommitted` | Possible | Possible | Possible | No shared locks acquired; reads do not block and are not blocked |
| `ReadCommitted` (default) | Prevented | Possible | Possible | Shared locks held only during the read of each row |
| `RepeatableRead` | Prevented | Prevented | Possible | Shared locks held until the end of the transaction |
| `Serializable` | Prevented | Prevented | Prevented | Range locks prevent inserts into ranges read by the transaction |
| `Snapshot` | Prevented | Prevented | Prevented | Row versioning -- reads see a consistent snapshot, no locks |

```csharp
// Specify isolation level when beginning the transaction
using var tx = conn.BeginTransaction(IsolationLevel.ReadCommitted); // default

// More restrictive -- prevent non-repeatable reads
using var tx2 = conn.BeginTransaction(IsolationLevel.RepeatableRead);

// Least restrictive -- allow dirty reads (fast but dangerous)
using var tx3 = conn.BeginTransaction(IsolationLevel.ReadUncommitted);

// Snapshot isolation (SQL Server -- requires database-level configuration)
using var tx4 = conn.BeginTransaction(IsolationLevel.Snapshot);
```

### Choosing the Right Level

| Use Case | Recommended Level | Why |
|---|---|---|
| Reporting / analytics (stale data OK) | `ReadUncommitted` or `Snapshot` | Maximum concurrency; no blocking |
| General OLTP (most applications) | `ReadCommitted` (default) | Good balance of consistency and performance |
| Financial calculations requiring consistent reads | `RepeatableRead` or `Snapshot` | Prevents mid-transaction data changes |
| Strict sequential processing | `Serializable` | Maximum isolation; prevents all anomalies |

```ad-warning
title: Serializable Can Cause Deadlocks
`Serializable` isolation uses range locks that can easily cause **deadlocks** when two transactions access overlapping ranges. Use it only when absolutely necessary, and keep transactions as short as possible. In most cases, `Snapshot` isolation achieves similar consistency with less blocking.
```

```ad-note
title: Snapshot Isolation Requires Database Configuration
Snapshot isolation uses **row versioning** in `tempdb` -- the database stores previous versions of modified rows so that readers see a consistent snapshot without acquiring locks. On SQL Server, you must first enable it:
```sql
ALTER DATABASE MyDatabase SET ALLOW_SNAPSHOT_ISOLATION ON;
```
This incurs additional `tempdb` overhead because every modified row has its previous version stored there.
```

---

## TransactionScope -- Ambient Transactions

`TransactionScope` (in `System.Transactions` namespace) provides a simpler, higher-level API for transactions. It creates an **ambient transaction** that automatically enlists any connection opened within its scope. It can also span **multiple connections** and even **multiple resource managers** (distributed transactions).

### Basic Usage

```csharp
using System.Transactions;

using var scope = new TransactionScope();

using var conn1 = new SqlConnection(connStr);
conn1.Open();
using var cmd1 = new SqlCommand("UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1", conn1);
cmd1.ExecuteNonQuery();

using var conn2 = new SqlConnection(connStr);
conn2.Open();
using var cmd2 = new SqlCommand("UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2", conn2);
cmd2.ExecuteNonQuery();

scope.Complete(); // signals that all operations succeeded
// Dispose() is called at the end of the using block:
// - If Complete() was called, the transaction COMMITS
// - If Complete() was NOT called (exception, early return), it ROLLS BACK
```

### With Async Code

```csharp
// CRITICAL: Must pass TransactionScopeAsyncFlowOption.Enabled for async
using var scope = new TransactionScope(
    TransactionScopeOption.Required,
    new TransactionOptions { IsolationLevel = IsolationLevel.ReadCommitted },
    TransactionScopeAsyncFlowOption.Enabled);

await using var conn = new SqlConnection(connStr);
await conn.OpenAsync();
// ... async operations ...
scope.Complete();
```

```ad-warning
title: TransactionScope and Async -- You MUST Pass TransactionScopeAsyncFlowOption.Enabled
Without `TransactionScopeAsyncFlowOption.Enabled`, the ambient transaction ==does not flow across `await` boundaries==. After the first `await`, the code resumes on a different thread that has no ambient transaction -- your commands execute outside the transaction silently. This is a very common and dangerous bug because there is no exception; operations just silently lose transactional protection.
```

### TransactionScopeOption

Controls how the scope interacts with an existing ambient transaction.

| Option | Behavior |
|---|---|
| `Required` (default) | Joins the existing ambient transaction if one exists; creates a new one if not |
| `RequiresNew` | Always creates a new transaction, suspending any existing ambient transaction |
| `Suppress` | Runs without any transaction, suppressing any existing ambient transaction |

```csharp
// Outer scope creates a transaction
using var outer = new TransactionScope();

// Inner scope joins the same transaction (Required)
using var inner = new TransactionScope(TransactionScopeOption.Required);
// ... operations are part of the outer transaction ...
inner.Complete();

// Or: inner scope creates an independent transaction (RequiresNew)
using var independent = new TransactionScope(TransactionScopeOption.RequiresNew);
// ... operations are in their own separate transaction ...
independent.Complete();

outer.Complete();
```

```ad-note
title: Distributed Transactions and Escalation
When a `TransactionScope` enlists more than one connection (or a non-SQL resource), the transaction **escalates** from a lightweight local transaction to a **distributed transaction** managed by the Microsoft Distributed Transaction Coordinator (MSDTC). Distributed transactions are significantly slower and require MSDTC to be configured and running. On .NET Core / .NET 5+, distributed transactions are **not supported on Linux** and require Windows.
```

---

## TransactionScope vs DbTransaction -- When to Use Which

| Aspect | `DbTransaction` | `TransactionScope` |
|---|---|---|
| API style | Explicit -- you call `BeginTransaction()`, `Commit()`, `Rollback()` | Implicit -- connections auto-enlist; `Complete()` or dispose |
| Scope | Single connection | Can span multiple connections / resource managers |
| Async support | Built-in | Requires `TransactionScopeAsyncFlowOption.Enabled` |
| Distributed transactions | No | Yes (escalates to MSDTC when needed) |
| Complexity | Low -- straightforward | Higher -- implicit behavior can be surprising |
| Recommended for | Single-connection CRUD operations | Service-layer transaction boundaries, multi-connection scenarios |

---

## Savepoints -- Partial Rollback Within a Transaction

Savepoints allow you to mark a point within a transaction and later **partially roll back** to that point without aborting the entire transaction. This is useful when part of the work is optional or can fail gracefully.

### SQL Server Savepoints

```csharp
using var conn = new SqlConnection(connStr);
conn.Open();
using var tx = conn.BeginTransaction();

try
{
    // Critical operation -- must succeed
    using var cmd1 = new SqlCommand(
        "INSERT INTO Orders (CustomerId, Total) VALUES (@CustId, @Total)", conn, tx);
    cmd1.Parameters.Add("@CustId", SqlDbType.Int).Value = 42;
    cmd1.Parameters.Add("@Total", SqlDbType.Decimal).Value = 99.99m;
    cmd1.ExecuteNonQuery();

    // Mark a savepoint before the optional operation
    tx.Save("BeforeNotification");

    try
    {
        // Optional operation -- log a notification (OK if it fails)
        using var cmd2 = new SqlCommand(
            "INSERT INTO Notifications (Message) VALUES (@Msg)", conn, tx);
        cmd2.Parameters.AddWithValue("@Msg", "New order placed");
        cmd2.ExecuteNonQuery();
    }
    catch
    {
        // Roll back only the notification insert; the order insert survives
        tx.Rollback("BeforeNotification");
        // Log the failure, but continue
    }

    tx.Commit(); // the order insert is committed regardless
}
catch
{
    tx.Rollback(); // full rollback on critical failure
    throw;
}
```

```ad-note
title: Savepoint Support Varies by Provider
Savepoints are well-supported in SQL Server (`DbTransaction.Save(string)`), PostgreSQL (`NpgsqlTransaction.CreateSavepoint(string)`), and MySQL. SQLite supports `SAVEPOINT` via raw SQL only. The `DbTransaction` base class does not define savepoint methods -- they are provider-specific.
```

---

## Common Pitfalls

### 1. Holding Transactions Open Too Long

```csharp
// BAD: Long-running transaction blocks other users
using var tx = conn.BeginTransaction();
using var cmd = new SqlCommand("SELECT * FROM Products", conn, tx);
using var reader = cmd.ExecuteReader();
while (reader.Read())
{
    // Slow operation per row -- holds locks for the entire duration
    Thread.Sleep(1000); // simulates expensive work
}
tx.Commit();
```

```ad-warning
title: Keep Transactions Short
Transactions hold locks on database resources. Long-running transactions increase contention, block other users, and raise the risk of deadlocks. Fetch data first, close the reader, then do expensive processing outside the transaction.
```

### 2. Forgetting to Dispose on Exception

```csharp
// RISKY: If an exception occurs between BeginTransaction and the try block,
// the transaction is never rolled back

// SAFER: use 'using' to ensure Dispose() rolls back uncommitted transactions
using var tx = conn.BeginTransaction();
// If the using block exits without Commit(), Dispose() rolls back automatically
```

### 3. Nested DbTransaction Is Not Supported

```csharp
// This throws InvalidOperationException:
// "SqlConnection does not support parallel transactions"
using var tx1 = conn.BeginTransaction();
using var tx2 = conn.BeginTransaction(); // THROWS
```

If you need nesting, use `TransactionScope` with `TransactionScopeOption.Required` or use savepoints.

---

## Summary

| Concept | Detail |
|---|---|
| What is a transaction | A group of operations that either all succeed (commit) or all fail (rollback) |
| ACID | Atomicity, Consistency, Isolation, Durability |
| Basic pattern | `BeginTransaction()` -> execute commands -> `Commit()` in try; `Rollback()` in catch |
| Command association | Every command must have `Transaction` set (constructor or property); forgetting throws |
| Isolation levels | `ReadUncommitted`, `ReadCommitted` (default), `RepeatableRead`, `Serializable`, `Snapshot` |
| `TransactionScope` | Ambient transactions; auto-enlists connections; supports distributed transactions |
| Async requirement | `TransactionScope` must use `TransactionScopeAsyncFlowOption.Enabled` with async code |
| Savepoints | Mark/rollback to a point within a transaction for partial rollback |
| Key rule | Keep transactions as short as possible to minimize lock contention |
