---
tags: [transactions, concurrency, acid]
---

- A **transaction** is a group of SQL operations that succeed or fail **together**. Partial completion is impossible — either every operation in the group commits, or none of them do. This is the foundation of data integrity in any serious database application.
- **Prerequisite:** [[01 - Query Optimization]], and the SQL Essentials folder (especially INSERT/UPDATE/DELETE). You need to understand data modification before you can understand what happens when modifications go wrong.

---

## Why Transactions Exist

- Consider a bank transfer: you need to debit one account and credit another. What happens if the system crashes after the debit but before the credit? Without transactions, the money vanishes — one account loses it, the other never receives it.
- Transactions guarantee that both operations happen, or neither does. They are the database's answer to the question: *"What happens if something goes wrong in the middle?"*

```sql
-- Without a transaction: DANGEROUS
UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- debit
-- *** server crashes here ***
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- credit never happens!

-- With a transaction: SAFE
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- debit
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- credit
COMMIT;  -- both happen, or both are rolled back
```

---

## ACID Properties

- Every transaction in a relational database guarantees four properties, known by the acronym **ACID**. These are not optional features — they are fundamental guarantees that the database engine enforces.

| Property | Meaning | What It Guarantees |
| --- | --- | --- |
| **Atomicity** | All or nothing | If any part of the transaction fails, ALL changes are rolled back. The database is never left in a half-finished state. |
| **Consistency** | Valid state to valid state | A transaction can only bring the database from one valid state to another. All constraints (foreign keys, CHECK, UNIQUE, NOT NULL) must be satisfied after the transaction completes. |
| **Isolation** | Concurrent transactions don't interfere | Even when multiple transactions run simultaneously, each one behaves as if it were the only one. The degree of isolation is configurable (see isolation levels below). |
| **Durability** | Committed data survives crashes | Once `COMMIT` returns successfully, the changes are permanently recorded — even if the server loses power one millisecond later. This is guaranteed by writing to the transaction log before confirming the commit. |

```ad-note
**Consistency** in ACID is different from "consistency" in distributed systems (CAP theorem). ACID consistency means constraint enforcement within a single database. CAP consistency means all nodes in a distributed system see the same data at the same time. Don't confuse the two — they solve different problems.
```

### How Durability Works — Write-Ahead Logging (WAL)

- The database doesn't write changed data pages to disk on every commit — that would be too slow (random I/O). Instead, it uses **Write-Ahead Logging**:
  1. Before modifying any data page, write a **log record** describing the change to the transaction log (sequential I/O — fast).
  2. The `COMMIT` command forces the log records to disk (fsync).
  3. The actual data pages are written to disk later, in the background (**checkpoint**).
- If the server crashes before a checkpoint, the database replays the transaction log on startup to redo committed changes and undo uncommitted ones. This is called **crash recovery**.

```ad-important
This is why the transaction log file is critical to database operation. If the log file runs out of space, the database cannot process any more transactions. Monitor log file size and back up the transaction log regularly (see [[03 - Backup and Recovery]]).
```

---

## Transaction Syntax

### Basic Transaction

```sql
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

- `BEGIN TRANSACTION` (or `BEGIN TRAN`) starts the transaction.
- `COMMIT` saves all changes permanently.
- `ROLLBACK` undoes all changes since `BEGIN TRANSACTION`.

### Manual ROLLBACK with Error Checking

```sql
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

IF @@ERROR = 0
    COMMIT;
ELSE
    ROLLBACK;
```

- `@@ERROR` returns the error number of the last statement. 0 means success.

```ad-warning
`@@ERROR` resets after every statement. If you check it after two statements, you only see the error from the *second* one. The TRY/CATCH pattern (below) is much more reliable and is the modern best practice.
```

### TRY/CATCH with Transactions (SQL Server — Best Practice)

```sql
BEGIN TRY
    BEGIN TRANSACTION;

    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;

    -- Both succeeded — commit
    COMMIT;
END TRY
BEGIN CATCH
    -- Something failed — undo everything
    IF @@TRANCOUNT > 0
        ROLLBACK;

    -- Re-raise the error so the caller knows
    THROW;
END CATCH
```

- `@@TRANCOUNT` returns the nesting level of transactions. Checking it before `ROLLBACK` prevents errors if the transaction was already rolled back by the engine (e.g., due to a deadlock).
- `THROW` re-raises the original error with the same message, severity, and state. This is better than `RAISERROR` for rethrowing caught errors.

```ad-note
Always put `BEGIN TRANSACTION` *inside* the `TRY` block, not before it. If you start the transaction outside the TRY and the `BEGIN TRANSACTION` itself fails (rare but possible), the CATCH block would try to roll back a transaction that doesn't exist.
```

### SAVEPOINT — Partial Rollbacks

- A **savepoint** marks a point within a transaction that you can roll back to without undoing the entire transaction:

```sql
BEGIN TRANSACTION;

INSERT INTO audit_log (action) VALUES ('transfer started');

SAVE TRANSACTION before_transfer;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Something went wrong with the transfer?
ROLLBACK TRANSACTION before_transfer;
-- Only the two UPDATEs are undone; the INSERT into audit_log is preserved

-- You can now try a different approach or just commit what's left
COMMIT;
```

- Savepoints are useful in complex stored procedures where you want to undo a specific section without aborting everything.

---

## Implicit vs Explicit Transactions

### Autocommit Mode (Default)

- By default, every single SQL statement is its own transaction. The database wraps each statement in an invisible `BEGIN TRAN ... COMMIT`:

```sql
-- These are THREE separate transactions:
INSERT INTO orders VALUES (1, '2026-06-01', 100);   -- auto-committed
INSERT INTO orders VALUES (2, '2026-06-02', 200);   -- auto-committed
INSERT INTO orders VALUES (3, '2026-06-03', 300);   -- auto-committed
```

- If the second INSERT fails, the first one is already committed and the third never runs. There is no automatic rollback of the group.

### Explicit Transactions

- When you use `BEGIN TRANSACTION`, you switch to **explicit transaction mode**. Nothing is committed until you say `COMMIT`. If any statement fails (and you have TRY/CATCH), you can roll back everything.

### Implicit Transaction Mode (SQL Server)

```sql
SET IMPLICIT_TRANSACTIONS ON;
```

- In this mode, SQL Server automatically starts a transaction before certain statements (SELECT, INSERT, UPDATE, DELETE, etc.) but does **not** automatically commit. You must explicitly `COMMIT` or `ROLLBACK`.

```ad-warning
`SET IMPLICIT_TRANSACTIONS ON` is a common source of long-running open transactions that hold locks and block other users. Avoid this setting unless you have a specific reason. Stick with autocommit (default) plus explicit `BEGIN TRANSACTION` when you need transactional groups.
```

---

## Concurrency Problems

- When multiple transactions run at the same time, they can interfere with each other in predictable ways. These are the four classic concurrency problems:

### Dirty Read

- **What happens:** Transaction A reads data that Transaction B has modified but not yet committed. If B rolls back, A has read data that never actually existed.

```
Transaction A                    Transaction B
                                 BEGIN TRAN
                                 UPDATE accounts SET balance = 0 WHERE id = 1;
                                 -- balance is now 0 (uncommitted)
SELECT balance FROM accounts
WHERE id = 1;
-- Reads 0 (dirty data!)
                                 ROLLBACK;
                                 -- balance is back to original value
-- A's data is now WRONG
```

### Non-Repeatable Read

- **What happens:** Transaction A reads the same row twice, but gets different values because Transaction B modified and committed it between the two reads.

```
Transaction A                    Transaction B
BEGIN TRAN
SELECT balance FROM accounts
WHERE id = 1;
-- Returns 1000
                                 UPDATE accounts SET balance = 500 WHERE id = 1;
                                 COMMIT;
SELECT balance FROM accounts
WHERE id = 1;
-- Returns 500 ← different!
COMMIT;
```

### Phantom Read

- **What happens:** Transaction A runs the same query twice, but the second time there are new rows (or missing rows) because Transaction B inserted or deleted rows that match A's filter.

```
Transaction A                    Transaction B
BEGIN TRAN
SELECT COUNT(*) FROM orders
WHERE status = 'pending';
-- Returns 10
                                 INSERT INTO orders (status) VALUES ('pending');
                                 COMMIT;
SELECT COUNT(*) FROM orders
WHERE status = 'pending';
-- Returns 11 ← phantom row!
COMMIT;
```

- The difference between non-repeatable reads and phantoms: non-repeatable reads involve **existing rows changing**, phantoms involve **new rows appearing or existing rows disappearing**.

### Lost Update

- **What happens:** Two transactions read the same data, then both update it based on what they read. The second write overwrites the first, and the first update is silently lost.

```
Transaction A                    Transaction B
SELECT balance FROM accounts
WHERE id = 1;
-- Returns 1000
                                 SELECT balance FROM accounts
                                 WHERE id = 1;
                                 -- Returns 1000
UPDATE accounts
SET balance = 1000 + 100         -- add 100
WHERE id = 1;
COMMIT;
                                 UPDATE accounts
                                 SET balance = 1000 - 200  -- subtract 200
                                 WHERE id = 1;
                                 COMMIT;
-- Final balance: 800
-- Should be 900 (1000 + 100 - 200)
-- A's +100 update is LOST
```

```ad-important
Lost updates are the most dangerous concurrency problem because they cause **silent data corruption**. The data looks valid — there's no error, no crash — but the numbers are wrong. This is especially critical in financial systems.
```

---

## Isolation Levels

- **Isolation levels** control how much protection a transaction gets from the concurrency problems above. Higher isolation = more protection but worse performance (more locking, less concurrency). Lower isolation = better performance but more risk of dirty/phantom reads.

| Isolation Level | Dirty Reads | Non-Repeatable Reads | Phantom Reads | Lost Updates | Performance Impact |
| --- | --- | --- | --- | --- | --- |
| **READ UNCOMMITTED** | Possible | Possible | Possible | Possible | Lowest (no shared locks) |
| **READ COMMITTED** | Prevented | Possible | Possible | Possible | Low (default in SQL Server) |
| **REPEATABLE READ** | Prevented | Prevented | Possible | Prevented | Medium |
| **SERIALIZABLE** | Prevented | Prevented | Prevented | Prevented | Highest (range locks) |
| **SNAPSHOT** | Prevented | Prevented | Prevented | Prevented | Low (row versioning, not locks) |

### Setting the Isolation Level

```sql
-- Set for the current session:
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Or for a specific query (SQL Server):
SELECT * FROM orders WITH (NOLOCK);  -- equivalent to READ UNCOMMITTED for this table
```

### READ UNCOMMITTED

- No shared locks are taken. Transactions can read uncommitted data from other transactions (dirty reads).
- **Use case:** Approximate counts, monitoring dashboards, reports where 100% accuracy is not critical and performance is paramount.

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT COUNT(*) FROM orders WHERE status = 'pending';
-- Fast, but the count might include orders from uncommitted transactions
```

```ad-warning
`WITH (NOLOCK)` and `READ UNCOMMITTED` are often used as a "go-faster" switch, but they can return **incorrect data** — not just stale data, but rows that never existed (from rolled-back transactions) or duplicate/missing rows (from page splits during the scan). Use only when approximate results are acceptable.
```

### READ COMMITTED (Default)

- Shared locks are taken while reading and released immediately after the read. This prevents dirty reads but not non-repeatable reads.
- **This is the default in SQL Server.** Most applications run at this level.

### REPEATABLE READ

- Shared locks are taken while reading and held until the end of the transaction. No other transaction can modify rows you've read until you commit or roll back.
- Prevents dirty reads and non-repeatable reads, but not phantom reads (new rows can still appear).

### SERIALIZABLE

- The strongest isolation level. Range locks prevent other transactions from inserting rows that would match your query's WHERE clause.
- Prevents all concurrency problems but significantly reduces concurrency — transactions effectively run one at a time.

```ad-warning
SERIALIZABLE dramatically increases the chance of deadlocks and lock wait timeouts. Use it only when absolute correctness is required for a specific, short-running transaction. Never set it as the session-wide default for a busy application.
```

### SNAPSHOT Isolation (SQL Server)

- Uses **row versioning** instead of locks. When a transaction starts, it sees a consistent snapshot of the database as of that moment. Other transactions can modify data freely — the snapshot transaction sees the old versions.
- Provides the same guarantees as SERIALIZABLE but without the locking overhead. The tradeoff: it requires more tempdb space to store row versions.

```sql
-- Must enable at the database level first (one-time setup):
ALTER DATABASE MyDb SET ALLOW_SNAPSHOT_ISOLATION ON;

-- Then use in a transaction:
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRANSACTION;
SELECT * FROM orders;  -- sees data as of the transaction start
-- Even if other transactions modify orders, this query returns the same results
COMMIT;
```

```ad-note
**READ COMMITTED SNAPSHOT ISOLATION (RCSI)** is a related feature that makes `READ COMMITTED` use row versioning instead of shared locks. It's enabled at the database level and changes the default behavior for all `READ COMMITTED` transactions. Many production SQL Server databases use RCSI because it dramatically reduces blocking without requiring any code changes.
```

```sql
-- Enable RCSI (one-time, requires exclusive access to the database):
ALTER DATABASE MyDb SET READ_COMMITTED_SNAPSHOT ON;
```

---

## Locks and Locking

- The database uses **locks** to enforce isolation levels. Understanding locks helps you diagnose blocking and deadlocks.

### Lock Types

| Lock Type | Abbreviation | When Taken | Compatibility |
| --- | --- | --- | --- |
| **Shared (S)** | S | Reading data (SELECT) | Compatible with other shared locks; blocks exclusive locks |
| **Exclusive (X)** | X | Modifying data (INSERT, UPDATE, DELETE) | Incompatible with everything — only one exclusive lock at a time |
| **Update (U)** | U | Reading data that *might* be updated | Compatible with shared locks; blocks other update and exclusive locks |
| **Intent locks** | IS, IX, IU | Signals at a higher level (table/page) that a lock exists at a lower level (row/page) | Prevents lock escalation conflicts |

### Lock Granularity

- Locks can be taken at different levels of granularity:

| Level | What's Locked | Concurrency | Overhead |
| --- | --- | --- | --- |
| **Row** | A single row | Highest concurrency | Highest memory overhead (one lock per row) |
| **Page** | An 8KB page (typically 10-100 rows) | Medium | Medium |
| **Table** | The entire table | Lowest concurrency | Lowest overhead (one lock) |

- The database engine automatically chooses the lock granularity. It starts with row locks and may **escalate** to a table lock if a single transaction holds too many row locks (default threshold: 5,000 row locks in SQL Server).

```ad-note
Lock escalation is a common cause of unexpected blocking. A large UPDATE or DELETE that touches thousands of rows can escalate to a table lock, blocking all other access to the table. To avoid this, process large changes in batches:
```

```sql
-- BAD: updates 500,000 rows → likely escalates to table lock
UPDATE orders SET status = 'archived' WHERE order_date < '2020-01-01';

-- GOOD: process in batches of 5,000 → stays with row locks
WHILE 1 = 1
BEGIN
    UPDATE TOP (5000) orders SET status = 'archived'
    WHERE order_date < '2020-01-01' AND status <> 'archived';

    IF @@ROWCOUNT = 0 BREAK;
END
```

### Viewing Locks

```sql
-- See current locks in SQL Server:
SELECT 
    resource_type, resource_description,
    request_mode, request_status,
    session_id
FROM sys.dm_tran_locks
WHERE resource_database_id = DB_ID('MyDb');

-- See who is blocking whom:
SELECT 
    blocking_session_id, session_id, wait_type, wait_time,
    text AS sql_text
FROM sys.dm_exec_requests
CROSS APPLY sys.dm_exec_sql_text(sql_handle)
WHERE blocking_session_id <> 0;
```

---

## Deadlocks

- A **deadlock** occurs when two (or more) transactions are each waiting for a lock held by the other. Neither can proceed — they are permanently stuck.

```
Transaction A                    Transaction B
BEGIN TRAN                       BEGIN TRAN
UPDATE table1 SET x = 1          UPDATE table2 SET y = 1
WHERE id = 1;                    WHERE id = 1;
-- Holds X lock on table1.id=1   -- Holds X lock on table2.id=1

UPDATE table2 SET y = 2          UPDATE table1 SET x = 2
WHERE id = 1;                    WHERE id = 1;
-- WAITS for B's lock            -- WAITS for A's lock
-- on table2.id=1                -- on table1.id=1

*** DEADLOCK — neither can proceed ***
```

### How the Database Resolves Deadlocks

- SQL Server runs a **deadlock monitor** that checks for deadlocks every few seconds. When it detects one, it chooses a **victim** — the transaction that is cheapest to roll back (based on the amount of log generated) — and kills it with error 1205.
- The victim's transaction is rolled back automatically. The other transaction proceeds.

### Preventing Deadlocks

1. **Access tables in the same order** — if all transactions lock table1 before table2, circular waits are impossible.
2. **Keep transactions short** — acquire locks, do the work, commit. Don't hold transactions open while waiting for user input or external API calls.
3. **Use the lowest necessary isolation level** — higher isolation = more locks = more deadlock risk.
4. **Add appropriate indexes** — without indexes, an UPDATE may scan (and lock) the entire table instead of just the target rows.
5. **Use SNAPSHOT isolation** — row versioning eliminates most lock conflicts entirely.
6. **Process large changes in batches** — reduces lock escalation and narrows the window for conflicts.

### Handling Deadlocks in Application Code

- Because deadlocks are sometimes unavoidable, your application should catch error 1205 and **retry the transaction**:

```csharp
// C# example — retry on deadlock
int retries = 3;
while (retries > 0)
{
    try
    {
        using var tran = connection.BeginTransaction();
        // ... execute commands ...
        tran.Commit();
        break;  // success — exit the loop
    }
    catch (SqlException ex) when (ex.Number == 1205)  // deadlock
    {
        retries--;
        if (retries == 0) throw;  // give up after 3 attempts
        // Optional: small delay before retry
    }
}
```

```ad-important
Deadlock retry logic belongs in the **application layer**, not in T-SQL. The application can log the event, apply backoff delays, and make intelligent decisions about whether to retry. In T-SQL, retry logic is clunky and limited.
```

---

## Transaction Best Practices

1. **Keep transactions as short as possible.** Start the transaction, do the work, commit. Never leave a transaction open while waiting for user input, API responses, or other external operations.
2. **Always use TRY/CATCH with explicit ROLLBACK.** Don't rely on the database to clean up after errors — be explicit.
3. **Don't nest transactions in SQL Server.** `BEGIN TRAN` inside another `BEGIN TRAN` increments `@@TRANCOUNT` but does not create a true nested transaction. Only the outermost `COMMIT` actually commits. A `ROLLBACK` anywhere rolls back *everything*. This is a common source of confusion.
4. **Use the appropriate isolation level.** `READ COMMITTED` (the default) is correct for most workloads. Escalate to `REPEATABLE READ` or `SNAPSHOT` only when you need specific guarantees.
5. **Avoid long-running transactions during business hours.** They hold locks that block other users and can cause log file growth.
6. **Monitor for open transactions.** A transaction left open by a crashed application holds locks indefinitely. Use `DBCC OPENTRAN` or query `sys.dm_tran_active_transactions` to find them.

```sql
-- Find the oldest open transaction:
DBCC OPENTRAN;

-- Or with more detail:
SELECT 
    s.session_id, s.login_name, s.host_name,
    t.transaction_begin_time,
    DATEDIFF(SECOND, t.transaction_begin_time, GETDATE()) AS age_seconds
FROM sys.dm_tran_active_transactions t
JOIN sys.dm_tran_session_transactions st ON t.transaction_id = st.transaction_id
JOIN sys.dm_exec_sessions s ON st.session_id = s.session_id
WHERE t.transaction_type = 1  -- user transaction
ORDER BY t.transaction_begin_time;
```

---

**Next:** [[03 - Backup and Recovery]]
