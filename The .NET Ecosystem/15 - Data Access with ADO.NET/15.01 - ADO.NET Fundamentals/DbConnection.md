---
tags:
  - csharp
  - ado-net
  - database-connections
aliases:
  - ADO.NET DbConnection
  - SqlConnection
  - Database Connection Lifecycle
---

## DbConnection

```ad-note
title: What You'll Learn
**`DbConnection`** is the abstract base class representing a connection to a database. Every ADO.NET operation starts with a connection. This note covers the connection lifecycle, the `ConnectionState` enum, synchronous and asynchronous usage, creating commands and transactions from a connection, best practices for connection management, and the relationship between connections and [[Connection Pooling|the connection pool]].
```

---

## Table of Contents

- [[#The DbConnection Class]]
- [[#Concrete Implementations]]
- [[#Connection Lifecycle]]
- [[#ConnectionState Enum]]
- [[#Key Properties]]
- [[#Key Methods]]
- [[#Async Connection Management]]
- [[#Creating Commands from a Connection]]
- [[#Transactions]]
- [[#IDisposable and Connection Pooling]]
- [[#Best Practices]]
- [[#Complete Working Examples]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## The DbConnection Class

`DbConnection` is defined in `System.Data.Common` and is the base for all database connections in ADO.NET:

```
System.Object
  └── System.MarshalByRefObject
       └── System.ComponentModel.Component        (IComponent, IDisposable)
            └── System.Data.Common.DbConnection    (IDbConnection, IAsyncDisposable)
                 ├── SqlConnection                  (Microsoft.Data.SqlClient)
                 ├── MySqlConnection                (MySqlConnector)
                 ├── NpgsqlConnection               (Npgsql)
                 └── SqliteConnection               (Microsoft.Data.Sqlite)
```

Key aspects:

- Inherits from `Component` — gains `IDisposable` and component model support
- Implements `IDbConnection` (legacy interface from .NET 1.0) and `IAsyncDisposable` (modern async disposal)
- You ==never instantiate `DbConnection` directly== — you create a concrete provider implementation
- Supports both synchronous and asynchronous operations

```ad-note
title: Section Summary
- `DbConnection` is the abstract base class in `System.Data.Common`
- All provider connections (`SqlConnection`, `MySqlConnection`, etc.) inherit from it
- It implements both `IDisposable` and `IAsyncDisposable`
```

---

## Concrete Implementations

| Provider | Connection Class | NuGet Package |
|---|---|---|
| SQL Server | `SqlConnection` | `Microsoft.Data.SqlClient` |
| MySQL / MariaDB | `MySqlConnection` | `MySqlConnector` |
| PostgreSQL | `NpgsqlConnection` | `Npgsql` |
| SQLite | `SqliteConnection` | `Microsoft.Data.Sqlite` |
| Oracle | `OracleConnection` | `Oracle.ManagedDataAccess.Core` |

```csharp
// Creating connections — each provider has the same pattern
using var sqlServerConn = new SqlConnection(connStr);
using var mysqlConn = new MySqlConnection(connStr);
using var pgConn = new NpgsqlConnection(connStr);
using var sqliteConn = new SqliteConnection("Data Source=mydb.db");
```

You can also program against the base type for [[Data Providers#The Provider Factory Pattern|provider-agnostic code]]:

```csharp
// Using the base type — works with any provider
DbConnection conn = new SqlConnection(connStr);  // ✅ upcast to base
await conn.OpenAsync();

// Or via factory
DbProviderFactory factory = SqlClientFactory.Instance;
DbConnection conn2 = factory.CreateConnection()!;
conn2.ConnectionString = connStr;
```

```ad-note
title: Section Summary
- Each provider has a concrete connection class that inherits from `DbConnection`
- You can program against the `DbConnection` base type for database-agnostic code
- Use `DbProviderFactory.CreateConnection()` when the provider is determined at runtime
```

---

## Connection Lifecycle

The connection follows a strict lifecycle that you must understand:

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│   ┌─────────┐   Set ConnectionString   ┌─────────────┐              │
│   │ Created  │ ──────────────────────► │ Configured  │              │
│   └─────────┘                          └──────┬──────┘              │
│       ▲                                       │                     │
│       │                               Open() / OpenAsync()          │
│       │                                       │                     │
│       │                                       ▼                     │
│       │                                ┌────────────┐               │
│       │                                │    Open    │               │
│       │                                │  (usable)  │               │
│       │                                └──────┬─────┘               │
│       │                                       │                     │
│       │                           Close() / Dispose() /             │
│       │                           DisposeAsync()                    │
│       │                                       │                     │
│       │                                       ▼                     │
│       │                                ┌────────────┐               │
│       │                                │   Closed   │──── if pool  │
│       │                                │            │    → returned │
│       │                                └──────┬─────┘   to pool    │
│       │                                       │                     │
│       │              Can re-open              │                     │
│       │◄──────────────────────────────────────┘                     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Step by Step

```csharp
// Step 1: Create — connection object exists but is not connected
var conn = new SqlConnection();
// conn.State == ConnectionState.Closed

// Step 2: Configure — set the connection string
conn.ConnectionString = "Server=localhost;Database=MyDb;Integrated Security=true";
// conn.State == ConnectionState.Closed (still)

// Step 3: Open — establish the connection (or get one from pool)
await conn.OpenAsync();
// conn.State == ConnectionState.Open

// Step 4: Use — execute commands, read data, manage transactions
using var cmd = conn.CreateCommand();
cmd.CommandText = "SELECT 1";
var result = await cmd.ExecuteScalarAsync();

// Step 5: Close/Dispose — return to pool
conn.Dispose(); // or conn.Close(); or end of using scope
// conn.State == ConnectionState.Closed
```

You can also combine steps 1 and 2 by passing the connection string to the constructor:

```csharp
// More concise — set connection string via constructor
using var conn = new SqlConnection("Server=localhost;Database=MyDb;Integrated Security=true");
await conn.OpenAsync();
```

```ad-warning
title: Don't Open Too Early
A common anti-pattern is opening the connection at the start of a method and doing non-database work before using it:

```csharp
// ❌ BAD — connection held open during non-database work
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();          // opened too early

var data = await httpClient.GetAsync(url);  // network call — 500ms
var processed = ProcessData(data);          // CPU work — 200ms

// Finally using the connection — 700ms after opening
using var cmd = conn.CreateCommand();
cmd.CommandText = "INSERT INTO ...";
await cmd.ExecuteNonQueryAsync();

// ✅ GOOD — open just before use
using var conn = new SqlConnection(connStr);

var data = await httpClient.GetAsync(url);
var processed = ProcessData(data);

await conn.OpenAsync();          // opened right before use
using var cmd = conn.CreateCommand();
cmd.CommandText = "INSERT INTO ...";
await cmd.ExecuteNonQueryAsync();
```

Open connections as ==late== as possible and close them as ==early== as possible. While the connection is open, it's checked out from the pool and unavailable to other requests.
```

```ad-note
title: Section Summary
- Lifecycle: Create → Configure → Open → Use → Close/Dispose
- Connections can be re-opened after closing
- Open as late as possible, close as early as possible
- Connection string can be set via constructor or property
```

---

## ConnectionState Enum

The `State` property returns a `ConnectionState` flags enum:

| Value | Numeric | Meaning |
|---|---|---|
| `Closed` | 0 | Connection is closed (initial state, after Close/Dispose) |
| `Open` | 1 | Connection is open and ready for use |
| `Connecting` | 2 | Connection is in the process of opening (rare to observe) |
| `Executing` | 4 | Connection is executing a command (provider-dependent) |
| `Fetching` | 8 | Connection is retrieving data (provider-dependent) |
| `Broken` | 16 | Connection was open but is now broken (network failure) |

```csharp
using var conn = new SqlConnection(connStr);

Console.WriteLine(conn.State); // Closed

await conn.OpenAsync();
Console.WriteLine(conn.State); // Open

// Check before using
if (conn.State != ConnectionState.Open)
{
    await conn.OpenAsync();
}
```

```ad-warning
title: Checking State is Not Thread-Safe
`conn.State` can change between the check and the use. In multi-threaded code, don't rely on state checks alone:

```csharp
// ❌ Race condition — state can change between check and use
if (conn.State == ConnectionState.Open)
{
    // Another thread could close it right here
    await cmd.ExecuteNonQueryAsync(); // might fail!
}

// ✅ Better — just try and handle the exception
try
{
    await cmd.ExecuteNonQueryAsync();
}
catch (InvalidOperationException ex) when (ex.Message.Contains("closed"))
{
    // Reconnect and retry
}
```

In practice, for most applications you should create a ==new connection per operation== rather than sharing a long-lived connection across threads. The pool makes this cheap.
```

```ad-info
title: Broken State
`ConnectionState.Broken` indicates the connection was Open but has been lost (e.g., network failure, server shutdown). A broken connection ==cannot be reopened== — you must close it and create a new one. The pool handles this automatically when you use the standard create-open-use-dispose pattern.
```

```ad-note
title: Section Summary
- `ConnectionState` has 6 values; you'll mostly see `Closed` and `Open`
- `Broken` means the connection was lost and must be recreated
- State checks are not thread-safe — prefer creating new connections per operation
```

---

## Key Properties

| Property | Type | Description |
|---|---|---|
| `ConnectionString` | `string` | The connection string used to open the connection. Can only be set when `State == Closed`. |
| `Database` | `string` | The current database name. Changes when `ChangeDatabase()` is called. |
| `DataSource` | `string` | The server name or address from the connection string. |
| `State` | `ConnectionState` | The current state of the connection. |
| `ServerVersion` | `string` | The version of the database server (only available when `State == Open`). |
| `ConnectionTimeout` | `int` | The time (seconds) to wait for a connection to open before throwing. Read-only; set via connection string. |

```csharp
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();

Console.WriteLine(conn.Database);          // "MyDb"
Console.WriteLine(conn.DataSource);        // "localhost"
Console.WriteLine(conn.State);             // Open
Console.WriteLine(conn.ServerVersion);     // "16.00.1000" (SQL Server 2022)
Console.WriteLine(conn.ConnectionTimeout); // 15 (default seconds)
```

```ad-warning
title: ServerVersion Throws When Closed
Accessing `ServerVersion` when the connection is closed throws `InvalidOperationException`. Always check `State == Open` or ensure you've called `Open()` first.
```

```ad-note
title: Section Summary
- `Database` and `DataSource` come from the connection string
- `ServerVersion` is only available when the connection is open
- `ConnectionTimeout` is read-only — set it in the connection string
```

---

## Key Methods

### Connection Management

| Method | Async Version | Description |
|---|---|---|
| `Open()` | `OpenAsync(CancellationToken)` | Opens the connection (or retrieves from pool) |
| `Close()` | `CloseAsync()` | Closes the connection (returns to pool) |
| `Dispose()` | `DisposeAsync()` | Closes and releases all resources |
| `ChangeDatabase(string)` | `ChangeDatabaseAsync(string)` | Switches to a different database on the same server |

### Factory Methods

| Method | Description |
|---|---|
| `CreateCommand()` | Creates a `DbCommand` associated with this connection |
| `BeginTransaction()` | Starts a new transaction on this connection |
| `BeginTransactionAsync()` | Async version of `BeginTransaction()` |
| `GetSchema()` | Returns schema information about the database |

### ChangeDatabase

```csharp
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();

Console.WriteLine(conn.Database); // "MyDb"

conn.ChangeDatabase("OtherDb");
Console.WriteLine(conn.Database); // "OtherDb"

// Equivalent to executing: USE [OtherDb]
// Useful in multi-database scenarios without opening a new connection
```

```ad-note
title: Section Summary
- `Open()`/`Close()` manage the connection lifecycle (all have async versions)
- `CreateCommand()` and `BeginTransaction()` are factory methods for creating associated objects
- `ChangeDatabase()` switches databases without a new connection (equivalent to SQL `USE`)
```

---

## Async Connection Management

In modern .NET applications (especially ASP.NET Core), ==always use the async methods==. Synchronous methods block the calling thread, wasting thread pool resources.

```csharp
// ✅ Async — releases the thread while waiting for the database
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();    // thread is free while establishing connection

using var cmd = conn.CreateCommand();
cmd.CommandText = "SELECT COUNT(*) FROM Users";
var count = (int)(await cmd.ExecuteScalarAsync())!;

// ✅ Async with CancellationToken — allows cancellation
using var conn2 = new SqlConnection(connStr);
await conn2.OpenAsync(cancellationToken);    // cancellable!

// ✅ Async dispose (C# 8+)
await using var conn3 = new SqlConnection(connStr);
await conn3.OpenAsync();
// ... use connection ...
// DisposeAsync() called at end of scope
```

```ad-warning
title: Synchronous Open() in ASP.NET Core
Using `conn.Open()` (synchronous) in an ASP.NET Core controller or service ==blocks a thread pool thread== while waiting for the connection. Under high load, this can exhaust the thread pool and starve the application. Always use `await conn.OpenAsync()` in async code paths.

```csharp
// ❌ In an async method — blocks a thread pool thread
public async Task<User> GetUserAsync(int id)
{
    using var conn = new SqlConnection(connStr);
    conn.Open();    // SYNC call in ASYNC method — thread blocked!
    // ...
}

// ✅ Fully async
public async Task<User> GetUserAsync(int id)
{
    using var conn = new SqlConnection(connStr);
    await conn.OpenAsync();    // thread released while waiting
    // ...
}
```
```

### `await using` for Async Disposal

`DbConnection` implements `IAsyncDisposable`, so you can use `await using`:

```csharp
// ✅ Best practice — async open AND async dispose
await using var conn = new SqlConnection(connStr);
await conn.OpenAsync();
// ... use connection ...
// DisposeAsync() called at end of scope — fully non-blocking
```

```ad-info
title: using vs await using
- `using var conn = ...` calls `Dispose()` (synchronous) at end of scope
- `await using var conn = ...` calls `DisposeAsync()` (asynchronous) at end of scope

Both work correctly. `await using` is slightly more efficient in async code because `DisposeAsync()` may involve async I/O. However, for most providers, `Dispose()` is already very fast (it just returns the connection to the pool), so the practical difference is minimal. Use `await using` when available, but don't lose sleep over `using` in async code.
```

```ad-note
title: Section Summary
- Always use `OpenAsync()` and `await using` in async code (ASP.NET Core, etc.)
- Synchronous `Open()` blocks thread pool threads and can exhaust them under load
- Pass `CancellationToken` to `OpenAsync()` for cancellable connection opening
- `await using` calls `DisposeAsync()` for fully non-blocking disposal
```

---

## Creating Commands from a Connection

The primary purpose of a connection is to create and execute commands. There are two patterns:

### Pattern 1: `CreateCommand()` (Provider-Agnostic)

```csharp
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();

// CreateCommand() returns a DbCommand pre-associated with this connection
using var cmd = conn.CreateCommand();
cmd.CommandText = "SELECT Id, Name FROM Users WHERE Active = @active";

// CreateParameter() also works — creates the correct parameter type
var param = cmd.CreateParameter();
param.ParameterName = "@active";
param.Value = true;
cmd.Parameters.Add(param);

using var reader = await cmd.ExecuteReaderAsync();
```

This pattern is ==fully database-agnostic== — no provider-specific types appear in the code. It works with `DbConnection`, so swapping providers requires no changes.

### Pattern 2: Provider-Specific Constructor

```csharp
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();

// SqlCommand constructor takes SQL + connection
using var cmd = new SqlCommand("SELECT Id, Name FROM Users WHERE Active = @active", conn);
cmd.Parameters.AddWithValue("@active", true);  // SqlParameter-specific convenience method

using var reader = await cmd.ExecuteReaderAsync();
```

This pattern is more concise but couples your code to a specific provider.

```ad-info
title: Which Pattern to Use?
- **Application code** that only targets one database: use the provider-specific constructor for brevity
- **Library code** or code that must support multiple databases: use `CreateCommand()` for agnosticism
- **With Dapper**: you typically pass the SQL and parameters to Dapper extension methods on the connection directly — Dapper handles command creation internally
```

```ad-note
title: Section Summary
- `CreateCommand()` creates a provider-agnostic `DbCommand` associated with the connection
- Provider-specific constructors (`new SqlCommand(sql, conn)`) are more concise but coupled
- Use `CreateCommand()` for database-agnostic code; use constructors for app-level code
```

---

## Transactions

`DbConnection` is the starting point for database transactions:

### Basic Transaction Pattern

```csharp
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();

// Begin a transaction on this connection
using var tx = await conn.BeginTransactionAsync();

try
{
    // All commands must be associated with the transaction
    using var cmd1 = conn.CreateCommand();
    cmd1.Transaction = (SqlTransaction)tx;   // associate command with transaction
    cmd1.CommandText = "UPDATE Accounts SET Balance = Balance - 100 WHERE Id = @from";
    cmd1.Parameters.AddWithValue("@from", 1);
    await cmd1.ExecuteNonQueryAsync();

    using var cmd2 = conn.CreateCommand();
    cmd2.Transaction = (SqlTransaction)tx;
    cmd2.CommandText = "UPDATE Accounts SET Balance = Balance + 100 WHERE Id = @to";
    cmd2.Parameters.AddWithValue("@to", 2);
    await cmd2.ExecuteNonQueryAsync();

    // Both succeeded — commit
    await tx.CommitAsync();
}
catch
{
    // Something failed — rollback
    await tx.RollbackAsync();
    throw;
}
```

### Transaction with Isolation Level

```csharp
// Specify isolation level when beginning the transaction
using var tx = await conn.BeginTransactionAsync(IsolationLevel.ReadCommitted);

// Common isolation levels:
// ReadUncommitted — dirty reads allowed (fastest, least consistent)
// ReadCommitted   — default for SQL Server; no dirty reads
// RepeatableRead  — no dirty or non-repeatable reads
// Serializable    — full isolation (slowest, most consistent)
// Snapshot        — SQL Server snapshot isolation (must be enabled on database)
```

```ad-warning
title: Command Must Reference the Transaction
Every `DbCommand` executed within a transaction ==must== have its `Transaction` property set. Forgetting this causes a runtime exception:

```csharp
// ❌ Throws: "Execute requires the command to have a transaction..."
using var tx = conn.BeginTransaction();
using var cmd = conn.CreateCommand();
cmd.CommandText = "INSERT INTO Log VALUES ('test')";
// cmd.Transaction = tx;  ← forgot this!
await cmd.ExecuteNonQueryAsync(); // EXCEPTION!

// ✅ Always set the Transaction property
using var tx = conn.BeginTransaction();
using var cmd = conn.CreateCommand();
cmd.Transaction = tx;   // required!
cmd.CommandText = "INSERT INTO Log VALUES ('test')";
await cmd.ExecuteNonQueryAsync();
```
```

```ad-info
title: Transaction Scope Alternative
For transactions that span multiple connections or multiple resource managers (e.g., database + message queue), consider `TransactionScope` from `System.Transactions`. However, `TransactionScope` can escalate to a distributed transaction (MSDTC), which has significant overhead. For single-connection transactions, `DbTransaction` is simpler and more efficient.
```

```ad-note
title: Section Summary
- Begin transactions via `conn.BeginTransaction()` or `conn.BeginTransactionAsync()`
- Every command in the transaction must set `cmd.Transaction = tx`
- Always use try/catch with `Commit()` in try and `Rollback()` in catch
- Specify isolation level when needed (default is `ReadCommitted` for SQL Server)
```

---

## IDisposable and Connection Pooling

`DbConnection` implements both `IDisposable` and `IAsyncDisposable`. Understanding what disposal does is critical:

```csharp
// When you call Dispose() on a connection:
// 1. If a transaction is active, it is ROLLED BACK (not committed!)
// 2. The connection state changes to Closed
// 3. If pooling is enabled: the physical connection is RETURNED TO THE POOL (not closed)
// 4. If pooling is disabled: the physical connection is actually closed
```

```ad-warning
title: Dispose Rolls Back Uncommitted Transactions
If you dispose a connection with an uncommitted transaction, the transaction is ==rolled back silently==. No exception, no warning. This is usually the correct behavior (if you didn't commit, something went wrong), but be aware of it:

```csharp
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();

using var tx = conn.BeginTransaction();
using var cmd = conn.CreateCommand();
cmd.Transaction = tx;
cmd.CommandText = "INSERT INTO Users (Name) VALUES ('Alice')";
await cmd.ExecuteNonQueryAsync();

// Forgot to call tx.Commit()!
// When conn.Dispose() runs, the transaction is ROLLED BACK
// Alice is NOT inserted — silently lost!
```
```

### Connection Reuse After Dispose

After `Dispose()`, the connection object is in `Closed` state. You ==can== set a new connection string and re-open it, but this is unusual and not recommended:

```csharp
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();
// ... use ...
conn.Close();   // returned to pool

// Technically possible to re-open
await conn.OpenAsync();   // gets a (possibly different) connection from pool
// ... use again ...
// Dispose runs at end of using scope

// This pattern is unusual — prefer creating a new connection object for clarity
```

```ad-note
title: Section Summary
- `Dispose()` closes the connection and returns it to the pool
- Uncommitted transactions are rolled back on `Dispose()` (silently, no exception)
- Connections can be re-opened after closing, but creating a new object is clearer
```

---

## Best Practices

### 1. Open Late, Close Early

```csharp
// ✅ Open just before use
await using var conn = new SqlConnection(connStr);
// ... do non-database work first ...
await conn.OpenAsync();        // right before the query
var result = await QueryAsync(conn);
// conn disposed immediately after use
```

### 2. One Connection Per Operation

```csharp
// ✅ Preferred — each method gets its own connection from the pool
public async Task<User> GetUserAsync(int id)
{
    await using var conn = new SqlConnection(connStr);
    await conn.OpenAsync();
    // ... query and return
}

public async Task UpdateUserAsync(User user)
{
    await using var conn = new SqlConnection(connStr);
    await conn.OpenAsync();
    // ... update
}
```

### 3. Don't Hold Connections Across Await Points (Without Need)

```csharp
// ❌ BAD — connection held open during HTTP call
await using var conn = new SqlConnection(connStr);
await conn.OpenAsync();

var externalData = await httpClient.GetAsync(url);  // connection sitting idle!
// ... then use conn ...

// ✅ GOOD — release connection, get a new one later
string dataFromDb;
await using (var conn1 = new SqlConnection(connStr))
{
    await conn1.OpenAsync();
    dataFromDb = await GetDataAsync(conn1);
} // connection returned to pool

var externalData = await httpClient.GetAsync(url);  // no connection held

await using (var conn2 = new SqlConnection(connStr))
{
    await conn2.OpenAsync();
    await SaveResultAsync(conn2, dataFromDb, externalData);
} // connection returned to pool
```

### 4. Never Share Connections Across Threads

```csharp
// ❌ DANGEROUS — DbConnection is NOT thread-safe
var conn = new SqlConnection(connStr);
await conn.OpenAsync();

await Task.WhenAll(
    QueryUsersAsync(conn),     // thread 1 uses conn
    QueryOrdersAsync(conn)     // thread 2 uses conn — RACE CONDITION!
);

// ✅ SAFE — each task gets its own connection
await Task.WhenAll(
    QueryUsersAsync(connStr),  // creates its own connection internally
    QueryOrdersAsync(connStr)  // creates its own connection internally
);
```

### 5. Store the Connection String, Not the Connection

```csharp
// ❌ BAD — storing a connection as a field
public class UserRepository
{
    private readonly SqlConnection _conn; // long-lived connection — DON'T
}

// ✅ GOOD — storing the connection string, creating connections per operation
public class UserRepository
{
    private readonly string _connStr;

    public UserRepository(string connStr) => _connStr = connStr;

    public async Task<User?> GetByIdAsync(int id)
    {
        await using var conn = new SqlConnection(_connStr);
        await conn.OpenAsync();
        // ...
    }
}
```

```ad-note
title: Section Summary
- Open late, close early — minimize the time a connection is checked out from the pool
- One connection per operation — the pool makes this cheap
- Never share connections across threads — `DbConnection` is not thread-safe
- Store connection strings (not connections) as fields in repositories and services
```

---

## Complete Working Examples

### Example 1: Simple Query

```csharp
public async Task<List<User>> GetActiveUsersAsync(string connStr)
{
    var users = new List<User>();

    await using var conn = new SqlConnection(connStr);
    await conn.OpenAsync();

    await using var cmd = conn.CreateCommand();
    cmd.CommandText = "SELECT Id, Name, Email FROM Users WHERE Active = 1";

    await using var reader = await cmd.ExecuteReaderAsync();
    while (await reader.ReadAsync())
    {
        users.Add(new User
        {
            Id = reader.GetInt32(reader.GetOrdinal("Id")),
            Name = reader.GetString(reader.GetOrdinal("Name")),
            Email = reader.IsDBNull(reader.GetOrdinal("Email"))
                ? null
                : reader.GetString(reader.GetOrdinal("Email"))
        });
    }

    return users;
}
```

### Example 2: Insert with Parameters

```csharp
public async Task<int> InsertUserAsync(string connStr, string name, string email)
{
    await using var conn = new SqlConnection(connStr);
    await conn.OpenAsync();

    await using var cmd = conn.CreateCommand();
    cmd.CommandText = @"
        INSERT INTO Users (Name, Email, CreatedAt)
        VALUES (@name, @email, @createdAt);
        SELECT SCOPE_IDENTITY();";    -- returns the new Id

    cmd.Parameters.AddWithValue("@name", name);
    cmd.Parameters.AddWithValue("@email", (object?)email ?? DBNull.Value);
    cmd.Parameters.AddWithValue("@createdAt", DateTime.UtcNow);

    // ExecuteScalar returns the first column of the first row
    var newId = Convert.ToInt32(await cmd.ExecuteScalarAsync());
    return newId;
}
```

### Example 3: Transaction

```csharp
public async Task TransferFundsAsync(string connStr, int fromId, int toId, decimal amount)
{
    await using var conn = new SqlConnection(connStr);
    await conn.OpenAsync();

    await using var tx = (SqlTransaction)await conn.BeginTransactionAsync();

    try
    {
        // Debit source account
        await using var debitCmd = conn.CreateCommand();
        debitCmd.Transaction = tx;
        debitCmd.CommandText = "UPDATE Accounts SET Balance = Balance - @amount WHERE Id = @id AND Balance >= @amount";
        debitCmd.Parameters.AddWithValue("@amount", amount);
        debitCmd.Parameters.AddWithValue("@id", fromId);

        int rowsAffected = await debitCmd.ExecuteNonQueryAsync();
        if (rowsAffected == 0)
            throw new InvalidOperationException("Insufficient funds or account not found.");

        // Credit destination account
        await using var creditCmd = conn.CreateCommand();
        creditCmd.Transaction = tx;
        creditCmd.CommandText = "UPDATE Accounts SET Balance = Balance + @amount WHERE Id = @id";
        creditCmd.Parameters.AddWithValue("@amount", amount);
        creditCmd.Parameters.AddWithValue("@id", toId);

        rowsAffected = await creditCmd.ExecuteNonQueryAsync();
        if (rowsAffected == 0)
            throw new InvalidOperationException("Destination account not found.");

        await tx.CommitAsync();
    }
    catch
    {
        await tx.RollbackAsync();
        throw;
    }
}
```

```ad-note
title: Section Summary
- Always use `await using` for connections, commands, and readers
- Handle `DBNull` when reading nullable columns
- Use `SCOPE_IDENTITY()` (SQL Server) to get auto-generated IDs after insert
- Wrap multi-statement operations in transactions with proper commit/rollback
```

---

## Comprehensive Summary

```ad-important
title: Key Takeaways
**`DbConnection`** is the abstract base class for all database connections in ADO.NET. Concrete implementations (`SqlConnection`, `MySqlConnection`, `NpgsqlConnection`, etc.) are provided by [[Data Providers|database-specific providers]].

**Lifecycle**: Create → Set ConnectionString → `Open()`/`OpenAsync()` → Use → `Dispose()`/`DisposeAsync()`. The connection can be re-opened after closing, but creating a new object is preferred for clarity.

**Async is mandatory** in modern .NET: Always use `OpenAsync()`, `await using`, and `DisposeAsync()`. Synchronous calls block thread pool threads and can exhaust them under load.

**Connection pooling integration**: `Open()` retrieves from the pool (or creates new), `Dispose()` returns to the pool (not physically closed). This makes creating a new connection per operation ==cheap and recommended==. See [[Connection Pooling]] for details.

**Critical rules**:
1. ==Always use `using` / `await using`== — leaked connections exhaust the pool
2. Open late, close early — minimize hold time
3. One connection per operation — the pool makes this cheap
4. Never share connections across threads — `DbConnection` is not thread-safe
5. Store connection strings, not connection objects, as class fields
6. Every command in a transaction must set `cmd.Transaction = tx`
7. Uncommitted transactions are silently rolled back on `Dispose()`
```

---

## Related Topics

- [[ADO.NET Overview]] — where `DbConnection` fits in the architecture
- [[Data Providers]] — the concrete implementations of `DbConnection`
- [[Connection Strings]] — configuring what `DbConnection` connects to
- [[Connection Pooling]] — how connection reuse works behind `Open()`/`Dispose()`
- [[DbCommand]] — executing queries and stored procedures on a connection
- [[DbDataReader]] — reading results from a command
- [[Transactions in ADO.NET]] — `DbTransaction` in depth
- [[Parameters and SQL Injection]] — safe parameterized queries
- [[IDisposable and the Dispose Pattern]] — why `using` is essential
