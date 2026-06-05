---
tags:
 - csharp
 - ado-net
 - connected-layer
---

## DbCommand -- Represents a SQL Statement or Stored Procedure to Execute Against a Database

`DbCommand` is the **abstract base class** in the `System.Data.Common` namespace that encapsulates a SQL statement, stored procedure call, or table-direct access to be executed against a data source. Every ADO.NET data provider supplies a concrete implementation:

| Provider | Concrete Class | Namespace |
|---|---|---|
| SQL Server | `SqlCommand` | `System.Data.SqlClient` / `Microsoft.Data.SqlClient` |
| MySQL | `MySqlCommand` | `MySqlConnector` / `MySql.Data.MySqlClient` |
| PostgreSQL | `NpgsqlCommand` | `Npgsql` |
| SQLite | `SqliteCommand` | `Microsoft.Data.Sqlite` |
| OLE DB | `OleDbCommand` | `System.Data.OleDb` |
| ODBC | `OdbcCommand` | `System.Data.Odbc` |

Because all of these derive from `DbCommand`, you can write **provider-agnostic** code by programming against the base class or the `IDbCommand` interface.

---

## Creating a DbCommand

There are two primary patterns for creating a command object.

### Direct Instantiation (Provider-Specific)

```csharp
using var cmd = new SqlCommand("SELECT Id, Name FROM Users", conn);
```

This ties your code to a specific provider (`SqlCommand`), but is the most common approach in applications that target a single database.

### Factory Method via DbConnection.CreateCommand() (Provider-Agnostic)

```csharp
using var cmd = conn.CreateCommand();
cmd.CommandText = "SELECT Id, Name FROM Users";
```

This is the **preferred approach** when writing provider-agnostic data access layers. The connection object creates the correct command type for its provider automatically.

```ad-note
title: DbProviderFactory Pattern
For fully provider-agnostic code, you can also use `DbProviderFactory.CreateCommand()`. This pairs with [[DbConnection]] obtained from the same factory, ensuring all objects belong to the same provider.
```

---

## Key Properties

| Property | Type | Description |
|---|---|---|
| `CommandText` | `string` | The SQL statement, stored procedure name, or table name to execute. |
| `Connection` | `DbConnection` | The connection object to execute against. Must be open before execution. |
| `Transaction` | `DbTransaction` | The transaction within which the command executes. `null` if no transaction is active. |
| `CommandType` | `CommandType` | Determines how `CommandText` is interpreted. Default is `Text`. |
| `CommandTimeout` | `int` | Seconds to wait before terminating an execution attempt. Default is **30 seconds**. Set to `0` for infinite wait. |
| `Parameters` | `DbParameterCollection` | The collection of parameters bound to this command. See [[Parameterized Queries]]. |
| `UpdatedRowSource` | `UpdateRowSource` | Determines how command results are applied to a `DataRow` when used by a `DbDataAdapter`. |
| `DesignTimeVisible` | `bool` | Whether the command should be visible in a designer control. Rarely used directly. |

---

## CommandType Enum

The `CommandType` property controls how the database engine interprets `CommandText`.

| Value | Meaning | Example |
|---|---|---|
| `Text` (default) | Raw SQL string | `"SELECT * FROM Users WHERE Id = @Id"` |
| `StoredProcedure` | Name of a stored procedure | `"sp_GetUserById"` |
| `TableDirect` | Name of a table (returns all rows) | `"Users"` |

```csharp
// Text (default) -- raw SQL
cmd.CommandType = CommandType.Text;
cmd.CommandText = "SELECT * FROM Users WHERE Active = 1";

// StoredProcedure -- call a stored proc by name
cmd.CommandType = CommandType.StoredProcedure;
cmd.CommandText = "sp_GetActiveUsers";

// TableDirect -- returns all rows from the table
cmd.CommandType = CommandType.TableDirect;
cmd.CommandText = "Users";
```

```ad-warning
title: TableDirect Is Not Universally Supported
`CommandType.TableDirect` is only supported by the **OLE DB** provider. Using it with `SqlCommand`, `NpgsqlCommand`, or `MySqlCommand` throws a `NotSupportedException`. In practice, you will almost always use `Text` or `StoredProcedure`.
```

---

## The Three Execution Methods

`DbCommand` provides three methods for executing the command, each suited to a different kind of query.

| Method | Returns | Use When |
|---|---|---|
| `ExecuteReader()` | `DbDataReader` | `SELECT` queries -- you need to iterate through rows |
| `ExecuteNonQuery()` | `int` (rows affected) | `INSERT`, `UPDATE`, `DELETE` -- you need the count of affected rows |
| `ExecuteScalar()` | `object?` (first column of first row) | Aggregate queries (`COUNT`, `SUM`, `MAX`) -- you need a single value |

### ExecuteReader()

Returns a [[DbDataReader]] -- a fast, forward-only, read-only stream of rows from the database. The connection remains **busy** until the reader is closed or disposed.

```csharp
using var conn = new SqlConnection(connStr);
conn.Open();

using var cmd = new SqlCommand("SELECT Id, Name, Email FROM Users WHERE Active = 1", conn);
using var reader = cmd.ExecuteReader();

while (reader.Read())
{
    int id       = reader.GetInt32(0);
    string name  = reader.GetString(1);
    string email = reader.GetString(2);
    Console.WriteLine($"{id}: {name} ({email})");
}
```

You can pass a `CommandBehavior` flag to control the reader's behavior:

```csharp
// CloseConnection -- the reader automatically closes the connection when it is disposed
using var reader = cmd.ExecuteReader(CommandBehavior.CloseConnection);
```

### ExecuteNonQuery()

Returns the number of rows affected by `INSERT`, `UPDATE`, or `DELETE` statements. For other statement types (e.g., `CREATE TABLE`, `ALTER`), it returns **-1**.

```csharp
using var cmd = new SqlCommand(
    "DELETE FROM Users WHERE LastLoginDate < @CutoffDate", conn);
cmd.Parameters.Add("@CutoffDate", SqlDbType.DateTime2).Value = DateTime.Now.AddYears(-2);

int rowsDeleted = cmd.ExecuteNonQuery();
Console.WriteLine($"Removed {rowsDeleted} inactive users.");
```

```ad-warning
title: DDL Statements Return -1
Statements that do not affect rows (like `CREATE TABLE`, `DROP INDEX`, `ALTER TABLE`) return `-1` from `ExecuteNonQuery()`, not `0`. Do not check the return value to determine success for DDL -- check for exceptions instead.
```

### ExecuteScalar()

Returns the **first column of the first row** in the result set. All other columns and rows are ignored. Returns `null` if the result set is empty.

```csharp
using var cmd = new SqlCommand("SELECT COUNT(*) FROM Users WHERE Active = 1", conn);
int activeCount = (int)cmd.ExecuteScalar()!;

// Another common pattern -- checking if a row exists
cmd.CommandText = "SELECT TOP 1 Id FROM Users WHERE Email = @Email";
cmd.Parameters.AddWithValue("@Email", "long@example.com");
object? result = cmd.ExecuteScalar();
bool exists = result != null;
```

```ad-note
title: ExecuteScalar and NULL
If the query returns a result set where the first column of the first row is SQL `NULL`, `ExecuteScalar()` returns `DBNull.Value` (not C# `null`). If the query returns **no rows at all**, it returns C# `null`. These are two different cases:
- **No rows**: `ExecuteScalar()` returns `null`
- **Row exists but value is NULL**: `ExecuteScalar()` returns `DBNull.Value`
```

---

## Async Execution Methods

Every execution method has an asynchronous counterpart that should be used in async contexts (ASP.NET Core, GUI applications, etc.) to avoid blocking threads.

| Synchronous | Asynchronous | Returns |
|---|---|---|
| `ExecuteReader()` | `ExecuteReaderAsync()` | `Task<DbDataReader>` |
| `ExecuteNonQuery()` | `ExecuteNonQueryAsync()` | `Task<int>` |
| `ExecuteScalar()` | `ExecuteScalarAsync()` | `Task<object?>` |

All async methods accept an optional `CancellationToken` parameter.

```csharp
using var cmd = new SqlCommand("SELECT Id, Name FROM Users", conn);

// Async reader with cancellation support
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(10));
await using var reader = await cmd.ExecuteReaderAsync(cts.Token);

while (await reader.ReadAsync(cts.Token))
{
    Console.WriteLine(reader.GetString(1));
}
```

```ad-important
title: Always Use Async in Server Applications
In ASP.NET Core and other server-side applications, ==always use the async execution methods==. Synchronous database calls block the thread pool thread for the entire duration of the query, which severely limits scalability under high load.
```

---

## Using DbCommand with Transactions

When executing commands inside a [[Transactions|transaction]], every command must be associated with the transaction object. Failing to do so throws an `InvalidOperationException`.

```csharp
using var conn = new SqlConnection(connStr);
conn.Open();
using var tx = conn.BeginTransaction();

try
{
    // Option 1: Pass transaction in constructor
    using var cmd1 = new SqlCommand(
        "UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1", conn, tx);
    cmd1.ExecuteNonQuery();

    // Option 2: Set Transaction property
    using var cmd2 = conn.CreateCommand();
    cmd2.Transaction = tx;
    cmd2.CommandText = "UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2";
    cmd2.ExecuteNonQuery();

    tx.Commit();
}
catch
{
    tx.Rollback();
    throw;
}
```

---

## CommandTimeout -- Controlling Query Timeouts

The `CommandTimeout` property controls how long (in seconds) the command waits for the database to respond before throwing a `TimeoutException` (or provider-specific equivalent like `SqlException`).

| Value | Behavior |
|---|---|
| `30` (default) | Wait up to 30 seconds |
| `0` | Wait indefinitely (use with extreme caution) |
| Any positive `int` | Wait that many seconds |

```csharp
// Long-running report query -- give it 5 minutes
using var cmd = new SqlCommand("EXEC sp_GenerateAnnualReport @Year", conn);
cmd.CommandTimeout = 300; // 5 minutes
cmd.Parameters.AddWithValue("@Year", 2025);
using var reader = cmd.ExecuteReader();
```

```ad-warning
title: CommandTimeout vs ConnectionTimeout
These are two **different** timeouts:
- `CommandTimeout` (on `DbCommand`) -- how long to wait for a **query** to complete
- `ConnectionTimeout` (on `DbConnection` / in the connection string) -- how long to wait to **establish a connection**

Setting `CommandTimeout = 0` means the command waits forever, which can cause your application to hang if the database is unresponsive.
```

---

## Prepare() -- Pre-compiling the Command

`DbCommand.Prepare()` tells the database to pre-compile the SQL statement, which can improve performance when executing the same parameterized command many times with different parameter values.

```csharp
using var cmd = new SqlCommand("INSERT INTO Logs (Message, Level) VALUES (@Msg, @Level)", conn);
cmd.Parameters.Add("@Msg", SqlDbType.NVarChar, 500);
cmd.Parameters.Add("@Level", SqlDbType.Int);
cmd.Prepare();  // pre-compile once

foreach (var log in logEntries)
{
    cmd.Parameters["@Msg"].Value = log.Message;
    cmd.Parameters["@Level"].Value = log.Level;
    cmd.ExecuteNonQuery();  // reuses the prepared plan
}
```

```ad-note
title: Prepare() Requires Defined Parameter Sizes
When calling `Prepare()`, you must define parameter sizes (e.g., `SqlDbType.NVarChar, 500`) in advance using `Add()`. Using `AddWithValue()` before `Prepare()` may cause issues because the size is inferred at runtime. Also note that SQL Server generally caches execution plans for parameterized queries automatically, so `Prepare()` provides less benefit there than on databases like PostgreSQL or MySQL.
```

---

## IDisposable -- Always Use `using`

`DbCommand` implements `IDisposable`. While disposing a command object does not close the connection, it releases internal resources held by the provider (prepared statement handles, etc.).

```csharp
// Preferred: using declaration
using var cmd = new SqlCommand("SELECT 1", conn);

// Or using block
using (var cmd2 = new SqlCommand("SELECT 1", conn))
{
    // ...
} // cmd2 disposed here
```

```ad-warning
title: Disposing the Command Does NOT Dispose the Reader
Disposing a `DbCommand` does **not** automatically close any open `DbDataReader` created from it. You must dispose the reader separately. Conversely, if you dispose the reader while the command is still alive, the command remains usable for another execution.
```

---

## Summary

| Concept | Detail |
|---|---|
| What it is | Abstract base class representing a SQL statement or stored procedure to execute |
| Concrete classes | `SqlCommand`, `MySqlCommand`, `NpgsqlCommand`, `SqliteCommand` |
| Key properties | `CommandText`, `Connection`, `Transaction`, `CommandType`, `CommandTimeout`, `Parameters` |
| `CommandType` values | `Text` (raw SQL), `StoredProcedure` (proc name), `TableDirect` (table name, OLE DB only) |
| `ExecuteReader()` | Returns a [[DbDataReader]] for `SELECT` queries |
| `ExecuteNonQuery()` | Returns rows affected for `INSERT`/`UPDATE`/`DELETE`; `-1` for DDL |
| `ExecuteScalar()` | Returns first column of first row; `null` if no rows, `DBNull.Value` if SQL NULL |
| Async variants | `ExecuteReaderAsync()`, `ExecuteNonQueryAsync()`, `ExecuteScalarAsync()` -- always prefer in server apps |
| `Prepare()` | Pre-compiles the statement; useful when executing the same query many times |
| Disposal | Always use `using`; does not close the connection or open readers |
