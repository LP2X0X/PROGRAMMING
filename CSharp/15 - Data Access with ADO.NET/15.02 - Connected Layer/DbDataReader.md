---
tags:
 - csharp
 - ado-net
 - connected-layer
---

## DbDataReader -- A Fast, Forward-Only, Read-Only Cursor Over Query Results

`DbDataReader` is the **abstract base class** in `System.Data.Common` that provides a high-performance, **forward-only**, **read-only** stream of rows from a database query. It is the fastest way to retrieve data from a database in .NET because it does not load the entire result set into memory -- it **streams** rows one at a time directly from the database server.

| Provider | Concrete Class |
|---|---|
| SQL Server | `SqlDataReader` |
| MySQL | `MySqlDataReader` |
| PostgreSQL | `NpgsqlDataReader` |
| SQLite | `SqliteDataReader` |
| OLE DB | `OleDbDataReader` |

---

## Why "Connected" -- The Reader Holds the Connection Open

The `DbDataReader` is the defining component of the **connected layer** of ADO.NET. While a reader is open:

- The [[DbConnection]] that produced it is **occupied** -- you cannot execute another command on the same connection (unless MARS is enabled on SQL Server).
- Data is being **streamed** from the database over the network -- it is not cached locally.
- If the connection is lost or closed, the reader becomes unusable.

This is in contrast to the **disconnected layer** ([[DataSet]], [[DataTable]]) where data is fetched in bulk and the connection is released immediately.

```ad-important
title: One Connection, One Reader (Without MARS)
By default, a connection can only serve **one active reader at a time**. Attempting to execute another command while a reader is open throws an `InvalidOperationException`: "There is already an open DataReader associated with this Connection which must be closed first." SQL Server supports **Multiple Active Result Sets (MARS)** via the connection string option `MultipleActiveResultSets=True`, but this adds overhead and should be used deliberately.
```

---

## Basic Reading Pattern

The fundamental pattern is: open connection, execute command, loop with `Read()`, close everything.

```csharp
using var conn = new SqlConnection(connStr);
conn.Open();

using var cmd = new SqlCommand("SELECT Id, Name, Age FROM Users", conn);
using var reader = cmd.ExecuteReader();

while (reader.Read())   // advances to next row; returns false when no more rows
{
    // Access by ordinal (column index) -- fastest
    int id       = reader.GetInt32(0);
    string name  = reader.GetString(1);
    int age      = reader.GetInt32(2);

    // Access by column name via indexer -- returns object, requires casting/conversion
    string name2 = reader["Name"].ToString()!;
    int age2     = (int)reader["Age"];

    Console.WriteLine($"{id}: {name}, age {age}");
}
```

```ad-note
title: Ordinal vs Name Access
Accessing columns by **ordinal** (integer index) is faster because it avoids a dictionary lookup. Accessing by **name** (string indexer) is more readable and resilient to column-order changes. A good middle ground is to resolve the ordinal once using `GetOrdinal()` and then use the ordinal for all rows:
```

```csharp
// Resolve ordinals once before the loop
int idOrd   = reader.GetOrdinal("Id");
int nameOrd = reader.GetOrdinal("Name");
int ageOrd  = reader.GetOrdinal("Age");

while (reader.Read())
{
    int id       = reader.GetInt32(idOrd);
    string name  = reader.GetString(nameOrd);
    int age      = reader.GetInt32(ageOrd);
}
```

---

## Key Methods and Properties

### Row Navigation

| Method / Property | Return Type | Description |
|---|---|---|
| `Read()` | `bool` | Advances the reader to the next row. Returns `false` when there are no more rows. Must be called before the first row. |
| `NextResult()` | `bool` | Advances to the next result set (for batch queries). Returns `false` if no more result sets. |
| `HasRows` | `bool` | `true` if the result set has at least one row. Does **not** advance the reader position. |
| `IsClosed` | `bool` | `true` if the reader has been closed or disposed. |
| `Close()` | `void` | Closes the reader and frees the connection for other commands. |
| `RecordsAffected` | `int` | Number of rows changed by `INSERT`/`UPDATE`/`DELETE` statements in the batch. `-1` for `SELECT`. |

### Typed Column Access (by Ordinal)

| Method | Return Type | Description |
|---|---|---|
| `GetInt32(ordinal)` | `int` | Returns the value as a 32-bit integer |
| `GetInt64(ordinal)` | `long` | Returns the value as a 64-bit integer |
| `GetString(ordinal)` | `string` | Returns the value as a string |
| `GetDateTime(ordinal)` | `DateTime` | Returns the value as a `DateTime` |
| `GetDecimal(ordinal)` | `decimal` | Returns the value as a decimal |
| `GetDouble(ordinal)` | `double` | Returns the value as a double-precision float |
| `GetBoolean(ordinal)` | `bool` | Returns the value as a boolean |
| `GetGuid(ordinal)` | `Guid` | Returns the value as a GUID |
| `GetByte(ordinal)` | `byte` | Returns the value as a single byte |
| `GetChar(ordinal)` | `char` | Returns the value as a single character |
| `GetValue(ordinal)` | `object` | Returns the value boxed as `object` |
| `GetValues(object[])` | `int` | Fills an array with all column values for the current row; returns column count |
| `GetFieldValue<T>(ordinal)` | `T` | **Generic typed access** -- the modern, preferred alternative to specific `GetXxx()` methods |

### Column Metadata

| Method / Property | Return Type | Description |
|---|---|---|
| `GetOrdinal(name)` | `int` | Returns the column index for a given column name. Throws if not found. |
| `GetName(ordinal)` | `string` | Returns the column name for a given index. |
| `GetDataTypeName(ordinal)` | `string` | Returns the database type name (e.g., `"nvarchar"`, `"int"`). |
| `GetFieldType(ordinal)` | `Type` | Returns the .NET `Type` of the column. |
| `FieldCount` | `int` | The number of columns in the current result set. |
| `IsDBNull(ordinal)` | `bool` | `true` if the column value is SQL `NULL`. |
| `GetSchemaTable()` | `DataTable?` | Returns detailed schema information for the result set columns. |

---

## Handling NULL Values

SQL `NULL` values require special care because calling a typed accessor like `GetString()` on a `NULL` column throws an `InvalidCastException`.

### The Safe Pattern -- Check IsDBNull() First

```csharp
while (reader.Read())
{
    int id      = reader.GetInt32(0);          // Id is NOT NULL in the schema
    string name = reader.GetString(1);         // Name is NOT NULL

    // Email might be NULL
    string? email = reader.IsDBNull(2) ? null : reader.GetString(2);

    // Age might be NULL -- map to Nullable<int>
    int? age = reader.IsDBNull(3) ? null : reader.GetInt32(3);
}
```

```ad-warning
title: GetString() on NULL Throws InvalidCastException
==Never call a typed accessor without checking `IsDBNull()` first== if the column is nullable. This is one of the most common ADO.NET bugs. The exception message ("Unable to cast object of type 'System.DBNull' to type 'System.String'") is not immediately obvious.
```

### Using GetFieldValue<T>() with Nullable Types

`GetFieldValue<T>()` is the generic alternative, but its behavior with `NULL` depends on the provider:

```csharp
// Some providers (e.g., Npgsql) support this directly:
string? email = reader.GetFieldValue<string?>(2);  // returns null for SQL NULL

// But SqlClient does NOT -- it throws on NULL just like GetString()
// Safe wrapper:
T? GetNullable<T>(DbDataReader r, int ordinal) where T : struct
    => r.IsDBNull(ordinal) ? null : r.GetFieldValue<T>(ordinal);

string? GetNullableString(DbDataReader r, int ordinal)
    => r.IsDBNull(ordinal) ? null : r.GetString(ordinal);
```

### The `as` Operator Pattern (Less Type-Safe)

```csharp
// Uses the object indexer -- returns DBNull.Value for NULL columns
string? email = reader["Email"] as string;    // returns null if DBNull
int? age      = reader["Age"] as int?;        // returns null if DBNull
```

---

## Async Reading

All critical methods have async counterparts that should be used in server-side code.

| Synchronous | Asynchronous | Returns |
|---|---|---|
| `Read()` | `ReadAsync(CancellationToken)` | `Task<bool>` |
| `NextResult()` | `NextResultAsync(CancellationToken)` | `Task<bool>` |
| `IsDBNull(ordinal)` | `IsDBNullAsync(ordinal, CancellationToken)` | `Task<bool>` |
| `GetFieldValue<T>(ordinal)` | `GetFieldValueAsync<T>(ordinal, CancellationToken)` | `Task<T>` |
| `Close()` | `CloseAsync()` | `Task` |
| `DisposeAsync()` | (via `IAsyncDisposable`) | `ValueTask` |

```csharp
await using var conn = new SqlConnection(connStr);
await conn.OpenAsync();

await using var cmd = new SqlCommand("SELECT Id, Name FROM Users", conn);
await using var reader = await cmd.ExecuteReaderAsync();

while (await reader.ReadAsync())
{
    int id      = reader.GetInt32(0);
    string name = reader.GetString(1);
    Console.WriteLine($"{id}: {name}");
}
```

```ad-note
title: await using for IAsyncDisposable
`DbDataReader` implements `IAsyncDisposable` (in modern providers). Use `await using` to ensure async disposal. If you use synchronous `using` in an async context, the `Dispose()` call may block.
```

---

## CommandBehavior Flags

When calling `ExecuteReader()`, you can pass `CommandBehavior` flags to control how the reader and connection behave.

| Flag | Effect |
|---|---|
| `Default` | No special behavior. |
| `CloseConnection` | The connection is automatically closed when the reader is closed/disposed. Useful when returning a reader from a method. |
| `SingleResult` | Only the first result set is processed; remaining result sets are discarded. Can improve performance. |
| `SingleRow` | Hints that only one row is expected. Some providers optimize for this. |
| `SequentialAccess` | Columns must be read in order. Required for streaming large binary/text columns (`BLOB`/`CLOB`). |
| `SchemaOnly` | Returns column metadata only; no data rows. Equivalent to `SET FMTONLY ON`. |
| `KeyInfo` | Appends primary key and unique column information to the schema. |

```csharp
// CloseConnection -- the reader owns the connection lifetime
using var reader = cmd.ExecuteReader(CommandBehavior.CloseConnection);
while (reader.Read()) { /* ... */ }
// reader.Dispose() also closes the connection

// SingleRow + SingleResult -- optimization for single-value lookups
using var reader2 = cmd.ExecuteReader(
    CommandBehavior.SingleResult | CommandBehavior.SingleRow);

// SequentialAccess -- for streaming large BLOBs
using var reader3 = cmd.ExecuteReader(CommandBehavior.SequentialAccess);
while (reader3.Read())
{
    // Must read columns in order: 0, then 1, then 2 -- cannot go back
    int id = reader3.GetInt32(0);
    // Stream a large binary column:
    using var stream = reader3.GetStream(1);  // column 1 is a VARBINARY(MAX)
    await stream.CopyToAsync(fileStream);
}
```

```ad-warning
title: SequentialAccess Restriction
With `SequentialAccess`, you **must** read columns in ascending ordinal order. Attempting to read column 0 after reading column 1 throws an `InvalidOperationException`. You also cannot read a column twice. This mode is essential for memory-efficient reading of large `VARBINARY(MAX)` or `NVARCHAR(MAX)` columns.
```

---

## Multiple Result Sets

A single command can contain multiple `SELECT` statements (a batch). The reader returns one result set at a time, and you advance to the next with `NextResult()`.

```csharp
cmd.CommandText = @"
    SELECT Id, Name FROM Users;
    SELECT Id, ProductName, Price FROM Products;
    SELECT COUNT(*) FROM Orders;";

using var reader = cmd.ExecuteReader();

// First result set: Users
Console.WriteLine("=== Users ===");
while (reader.Read())
{
    Console.WriteLine($"{reader.GetInt32(0)}: {reader.GetString(1)}");
}

// Advance to second result set: Products
if (reader.NextResult())
{
    Console.WriteLine("=== Products ===");
    while (reader.Read())
    {
        Console.WriteLine($"{reader.GetInt32(0)}: {reader.GetString(1)} - ${reader.GetDecimal(2)}");
    }
}

// Advance to third result set: Order count
if (reader.NextResult())
{
    reader.Read();
    int orderCount = reader.GetInt32(0);
    Console.WriteLine($"Total orders: {orderCount}");
}
```

```ad-note
title: Batch Queries Reduce Round Trips
Sending multiple queries in a single batch reduces network round trips, which can significantly improve performance over high-latency connections. Each `NextResult()` advances to the next result set without a new round trip.
```

---

## Performance Characteristics

`DbDataReader` is the **fastest** data access mechanism in .NET because of its design constraints:

| Characteristic | Implication |
|---|---|
| Forward-only | No `MovePrevious()` or random access -- forces single-pass processing |
| Read-only | No `Update()` or `Delete()` on the reader -- data is immutable |
| Streaming | Rows are not buffered in memory -- memory usage is O(1) regardless of result set size |
| Connected | Holds the connection open -- you must process rows quickly and close the reader |

### Comparison with Other Data Access Approaches

| Approach | Memory | Speed | Connection Usage |
|---|---|---|---|
| `DbDataReader` | O(1) -- streams rows | Fastest | Holds connection until closed |
| `DataTable.Load(reader)` | O(n) -- loads all rows | Slower (allocation overhead) | Releases after load |
| `DataAdapter.Fill(DataSet)` | O(n) -- loads all rows | Slower | Opens and closes automatically |
| Entity Framework | O(n) -- materializes objects | Slowest (change tracking overhead) | Releases after query |

```ad-important
title: Close the Reader Promptly
Because the reader holds the connection open, you should process rows as quickly as possible and close the reader. Avoid doing expensive per-row operations (HTTP calls, file I/O) inside the `while (reader.Read())` loop. If you need to do post-processing, load data into a `List<T>` first, close the reader, then process.
```

---

## Mapping Rows to Objects

A common pattern is to map reader rows into strongly-typed objects:

```csharp
public record User(int Id, string Name, string? Email, int? Age);

public static List<User> GetAllUsers(SqlConnection conn)
{
    using var cmd = new SqlCommand("SELECT Id, Name, Email, Age FROM Users", conn);
    using var reader = cmd.ExecuteReader();

    var users = new List<User>();

    int idOrd    = reader.GetOrdinal("Id");
    int nameOrd  = reader.GetOrdinal("Name");
    int emailOrd = reader.GetOrdinal("Email");
    int ageOrd   = reader.GetOrdinal("Age");

    while (reader.Read())
    {
        users.Add(new User(
            Id:    reader.GetInt32(idOrd),
            Name:  reader.GetString(nameOrd),
            Email: reader.IsDBNull(emailOrd) ? null : reader.GetString(emailOrd),
            Age:   reader.IsDBNull(ageOrd)   ? null : reader.GetInt32(ageOrd)
        ));
    }

    return users;
}
```

This is essentially what ORMs like [[Entity Framework]] and micro-ORMs like Dapper do behind the scenes -- Dapper in particular generates optimized IL code for this mapping at runtime.

---

## Summary

| Concept | Detail |
|---|---|
| What it is | Abstract base class for a fast, forward-only, read-only stream of database rows |
| Why "connected" | The reader holds the connection open while streaming data |
| Core loop | `while (reader.Read()) { ... }` -- call `Read()` before accessing the first row |
| Typed access | `GetInt32()`, `GetString()`, `GetDateTime()`, etc. by ordinal; or generic `GetFieldValue<T>()` |
| Name-based access | `reader["ColumnName"]` returns `object`; or resolve ordinal once with `GetOrdinal()` |
| NULL handling | Always check `IsDBNull()` before typed access; `GetString()` on NULL throws |
| Multiple result sets | Use `NextResult()` to advance to the next `SELECT` in a batch |
| `CommandBehavior` flags | `CloseConnection`, `SingleRow`, `SingleResult`, `SequentialAccess`, `SchemaOnly` |
| Async | `ReadAsync()`, `NextResultAsync()`, `GetFieldValueAsync<T>()`, `await using` for disposal |
| Performance | Fastest data retrieval in .NET -- O(1) memory, forward-only streaming |
