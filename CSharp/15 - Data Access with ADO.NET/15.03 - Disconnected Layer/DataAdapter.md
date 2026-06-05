---
tags:
  - csharp
  - ado-net
  - disconnected-layer
aliases:
  - DbDataAdapter
  - SqlDataAdapter
  - Data Adapter
  - CommandBuilder
  - SqlCommandBuilder
---

## DataAdapter

```ad-note
title: What You'll Learn
The `DbDataAdapter` is the ==bridge between the database and in-memory data structures== (`DataSet`/`DataTable`). It handles two directions of data flow: `Fill()` reads from the database into memory, and `Update()` pushes in-memory changes back to the database. This note covers the adapter's architecture, how `Fill()` and `Update()` work, automatic command generation with `DbCommandBuilder`, manual command configuration, batch updating, table mappings, and handling concurrency conflicts during updates.
```

---

## Table of Contents

- [[#Architecture — The Bridge Pattern]]
- [[#Fill — Database to Memory]]
  - [[#Basic Fill]]
  - [[#Fill with DataSet (Multiple Tables)]]
  - [[#Fill with Paging]]
  - [[#FillSchema — Importing Schema Only]]
- [[#Update — Memory to Database]]
  - [[#How Update Works Internally]]
  - [[#Update with CommandBuilder (Automatic Commands)]]
  - [[#Setting Commands Manually]]
  - [[#Parameter Source Columns]]
- [[#DbCommandBuilder — Automatic Command Generation]]
  - [[#How CommandBuilder Works]]
  - [[#Limitations]]
  - [[#Refreshing Commands]]
- [[#Batch Updating]]
- [[#Table Mappings]]
- [[#Handling Events — RowUpdating and RowUpdated]]
- [[#Concurrency Conflicts]]
- [[#Async Operations]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## Architecture — The Bridge Pattern

The `DbDataAdapter` sits between two worlds:

```
┌──────────────────┐         ┌──────────────────────────────────┐
│     Database     │ ◄─────► │         DbDataAdapter            │
│  (SQL Server,    │         │                                  │
│   MySQL, etc.)   │         │  SelectCommand ── Fill() ──────► │
│                  │         │  InsertCommand ◄── Update() ──── │
│                  │         │  UpdateCommand ◄── Update() ──── │
│                  │         │  DeleteCommand ◄── Update() ──── │
└──────────────────┘         └──────────────┬───────────────────┘
                                            │
                                            ▼
                             ┌──────────────────────────────────┐
                             │    DataSet / DataTable            │
                             │    (In-memory data)               │
                             └──────────────────────────────────┘
```

The adapter has **four command properties**:

| Property | Used By | Direction | Purpose |
|---|---|---|---|
| `SelectCommand` | `Fill()` | Database → Memory | Fetches rows from the database |
| `InsertCommand` | `Update()` | Memory → Database | Inserts rows where `RowState == Added` |
| `UpdateCommand` | `Update()` | Memory → Database | Updates rows where `RowState == Modified` |
| `DeleteCommand` | `Update()` | Memory → Database | Deletes rows where `RowState == Deleted` |

Each concrete data provider has its own adapter class:

| Provider | Adapter Class | NuGet Package |
|---|---|---|
| SQL Server | `SqlDataAdapter` | `Microsoft.Data.SqlClient` |
| MySQL / MariaDB | `MySqlDataAdapter` | `MySqlConnector` |
| PostgreSQL | `NpgsqlDataAdapter` | `Npgsql` |
| SQLite | `SqliteDataAdapter` | *(not provided — SQLite is rarely used with the disconnected layer)* |

```ad-note
title: Section Summary
- `DbDataAdapter` bridges the database and `DataSet`/`DataTable` using four command properties
- `Fill()` uses `SelectCommand`; `Update()` uses `InsertCommand`, `UpdateCommand`, `DeleteCommand`
- Each database provider supplies its own concrete adapter class
```

---

## Fill — Database to Memory

### Basic Fill

`Fill()` executes the `SelectCommand`, reads the results, and populates a `DataTable`:

```csharp
using var conn = new SqlConnection(connStr);
using var adapter = new SqlDataAdapter("SELECT Id, Name, Age FROM Users", conn);

var table = new DataTable();
int rowsAffected = adapter.Fill(table); // returns the number of rows added/refreshed
Console.WriteLine($"Loaded {rowsAffected} rows");

foreach (DataRow row in table.Rows)
{
    Console.WriteLine($"{row["Id"]}: {row["Name"]}, age {row["Age"]}");
}
```

```ad-important
title: Fill() Manages the Connection Automatically
`Fill()` has smart connection handling:
- If the connection was **closed**, `Fill()` opens it, reads data, and ==closes it automatically==
- If the connection was **already open**, `Fill()` leaves it open after finishing

This means you typically **do not need to call `conn.Open()` or `conn.Close()` yourself** when using `Fill()`. The same applies to `Update()`.
```

What `Fill()` does internally:

1. Opens the connection (if closed)
2. Executes `SelectCommand` to get a `DbDataReader`
3. Creates `DataColumn` objects from the reader's schema (if the table has no columns yet)
4. Reads all rows from the reader into `DataRow` objects
5. Sets every loaded row's `RowState` to `Unchanged`
6. Closes the connection (if it opened it)

```ad-warning
title: Fill() Does Not Clear Existing Rows
If the `DataTable` already has data, `Fill()` **adds to it** — it does not clear existing rows first. If the primary key matches, it merges (updates the existing row). If no primary key is defined, you get duplicate rows on repeated `Fill()` calls.

To get clean data on each fill, either:
- Call `table.Clear()` before `Fill()`
- Or ensure the table has a primary key set so matching rows are updated rather than duplicated
```

### Fill with DataSet (Multiple Tables)

You can fill a `DataSet` with a named table:

```csharp
var ds = new DataSet();
using var adapter = new SqlDataAdapter("SELECT Id, Name FROM Users", connStr);

// Fills ds.Tables["Users"] (creates the table if it doesn't exist)
adapter.Fill(ds, "Users");

// Fill another table with a different adapter
using var orderAdapter = new SqlDataAdapter("SELECT * FROM Orders", connStr);
orderAdapter.Fill(ds, "Orders");

// Access
DataTable users = ds.Tables["Users"]!;
DataTable orders = ds.Tables["Orders"]!;
```

Filling multiple result sets from a single query:

```csharp
// Stored procedure or batch query that returns multiple result sets
using var adapter = new SqlDataAdapter("SELECT * FROM Users; SELECT * FROM Orders;", connStr);

var ds = new DataSet();
adapter.Fill(ds);
// ds.Tables[0] = first result set (auto-named "Table")
// ds.Tables[1] = second result set (auto-named "Table1")

// Use TableMappings for better names (see Table Mappings section)
```

### Fill with Paging

`Fill()` supports paging — loading a subset of rows:

```csharp
// Fill(DataSet, startRecord, maxRecords, tableName)
adapter.Fill(ds, 0, 50, "Users");    // first 50 rows
adapter.Fill(ds, 50, 50, "Users");   // next 50 rows
```

```ad-warning
title: Fill Paging Is Inefficient
The paging overload of `Fill()` still executes the ==full query== on the database and fetches all rows — it just discards the ones outside the range. For real paging, use SQL-level paging (`OFFSET`/`FETCH`, `LIMIT`/`OFFSET`) in your `SelectCommand` instead.
```

### FillSchema — Importing Schema Only

`FillSchema()` reads the schema (column names, types, primary keys, constraints) without loading any data:

```csharp
var table = new DataTable();
adapter.FillSchema(table, SchemaType.Source);

// table.Columns are populated with types, MaxLength, AllowDBNull, etc.
// table.PrimaryKey is set
// table.Rows is empty — no data loaded
```

| SchemaType | Behavior |
|---|---|
| `Source` | Uses column names and types from the database exactly |
| `Mapped` | Applies `TableMappings` / `ColumnMappings` to rename columns |

```ad-note
title: Section Summary
- `Fill()` executes `SelectCommand`, reads all rows into a `DataTable`, sets `RowState` to `Unchanged`
- `Fill()` auto-manages the connection (opens if closed, closes after)
- Repeated `Fill()` on a table without `Clear()` or a primary key causes duplicate rows
- Fill-paging is inefficient — prefer SQL-level `OFFSET`/`FETCH` for real paging
- `FillSchema()` imports schema metadata without loading data
```

---

## Update — Memory to Database

### How Update Works Internally

`Update()` iterates through all rows in the `DataTable` and executes the appropriate command based on each row's `RowState`:

```
For each DataRow in table.Rows:
    switch (row.RowState)
    {
        case Added:     execute InsertCommand with row's Current values
        case Modified:  execute UpdateCommand with row's Current + Original values
        case Deleted:   execute DeleteCommand with row's Original values
        case Unchanged: skip
        case Detached:  skip
    }
    
    If command succeeds:
        call row.AcceptChanges()  // RowState → Unchanged (or row removed if Deleted)
```

```csharp
// Modify data
table.Rows[0]["Name"] = "Updated Name";                 // RowState → Modified
table.Rows.Add(3, "New Person", 30);                    // RowState → Added
table.Rows[1].Delete();                                 // RowState → Deleted

// Push changes (requires InsertCommand, UpdateCommand, DeleteCommand to be set)
int rowsAffected = adapter.Update(table);
Console.WriteLine($"Updated {rowsAffected} rows in the database");

// After Update(), successfully synced rows have RowState == Unchanged
```

### Update with CommandBuilder (Automatic Commands)

The simplest way to use `Update()` is with a `DbCommandBuilder`, which auto-generates the `INSERT`, `UPDATE`, and `DELETE` commands from the `SelectCommand`:

```csharp
using var conn = new SqlConnection(connStr);
using var adapter = new SqlDataAdapter("SELECT Id, Name, Age FROM Users", conn);

// Fill the table
var table = new DataTable();
adapter.Fill(table);

// Make modifications
table.Rows[0]["Name"] = "Updated";
table.Rows.Add(null, "New User", 25);  // null for auto-increment Id

// Create a CommandBuilder — it generates Insert/Update/Delete commands automatically
using var builder = new SqlCommandBuilder(adapter);

// Now adapter.InsertCommand, adapter.UpdateCommand, adapter.DeleteCommand are set
adapter.Update(table);
```

### Setting Commands Manually

For complex scenarios (stored procedures, custom SQL, joins), you set the commands yourself:

```csharp
using var conn = new SqlConnection(connStr);
using var adapter = new SqlDataAdapter("SELECT Id, Name, Age FROM Users", conn);

// InsertCommand
adapter.InsertCommand = new SqlCommand(
    "INSERT INTO Users (Name, Age) VALUES (@Name, @Age); SELECT SCOPE_IDENTITY();", conn);
adapter.InsertCommand.Parameters.Add("@Name", SqlDbType.NVarChar, 100, "Name");
adapter.InsertCommand.Parameters.Add("@Age", SqlDbType.Int, 0, "Age");

// UpdateCommand
adapter.UpdateCommand = new SqlCommand(
    "UPDATE Users SET Name = @Name, Age = @Age WHERE Id = @Id AND Name = @OrigName", conn);
adapter.UpdateCommand.Parameters.Add("@Name", SqlDbType.NVarChar, 100, "Name");
adapter.UpdateCommand.Parameters.Add("@Age", SqlDbType.Int, 0, "Age");
adapter.UpdateCommand.Parameters.Add("@Id", SqlDbType.Int, 0, "Id");
// Original version parameter for optimistic concurrency
var origName = adapter.UpdateCommand.Parameters.Add("@OrigName", SqlDbType.NVarChar, 100, "Name");
origName.SourceVersion = DataRowVersion.Original;

// DeleteCommand
adapter.DeleteCommand = new SqlCommand(
    "DELETE FROM Users WHERE Id = @Id", conn);
adapter.DeleteCommand.Parameters.Add("@Id", SqlDbType.Int, 0, "Id");
```

**Using stored procedures:**

```csharp
adapter.InsertCommand = new SqlCommand("sp_InsertUser", conn)
{
    CommandType = CommandType.StoredProcedure
};
adapter.InsertCommand.Parameters.Add("@Name", SqlDbType.NVarChar, 100, "Name");
adapter.InsertCommand.Parameters.Add("@Age", SqlDbType.Int, 0, "Age");
// Output parameter to capture the generated Id
var idParam = adapter.InsertCommand.Parameters.Add("@NewId", SqlDbType.Int);
idParam.Direction = ParameterDirection.Output;
idParam.SourceColumn = "Id";
```

### Parameter Source Columns

When you add a parameter to an adapter command, the **`SourceColumn`** property tells the adapter which `DataColumn` to read the value from:

```csharp
// The 4th argument in Add() is the SourceColumn
adapter.InsertCommand.Parameters.Add("@Name", SqlDbType.NVarChar, 100, "Name");
//                                                                      ^^^^^ 
//                                           reads from row["Name"] (Current version)
```

The **`SourceVersion`** property controls which row version the parameter reads:

| SourceVersion | Used For | Reads |
|---|---|---|
| `Current` (default) | `INSERT` and `UPDATE` new values | `row["Col", DataRowVersion.Current]` |
| `Original` | `UPDATE` / `DELETE` `WHERE` clauses | `row["Col", DataRowVersion.Original]` |

```csharp
// Optimistic concurrency: WHERE clause uses Original values
var param = new SqlParameter("@OriginalName", SqlDbType.NVarChar, 100)
{
    SourceColumn = "Name",
    SourceVersion = DataRowVersion.Original  // reads the value BEFORE modification
};
adapter.UpdateCommand.Parameters.Add(param);
```

```ad-note
title: Section Summary
- `Update()` iterates rows, executes the command matching each row's `RowState`, then calls `AcceptChanges()` on success
- `CommandBuilder` auto-generates commands for simple single-table queries
- For complex scenarios, set `InsertCommand`, `UpdateCommand`, `DeleteCommand` manually
- `SourceColumn` maps parameters to `DataColumn` names; `SourceVersion` controls which row version is read
- Use `SourceVersion = Original` for `WHERE` clause parameters (optimistic concurrency)
```

---

## DbCommandBuilder — Automatic Command Generation

### How CommandBuilder Works

`DbCommandBuilder` inspects the `SelectCommand` of the adapter and generates `INSERT`, `UPDATE`, and `DELETE` commands at runtime:

```csharp
using var adapter = new SqlDataAdapter("SELECT Id, Name, Age FROM Users", connStr);
using var builder = new SqlCommandBuilder(adapter);

// Examine the generated commands
Console.WriteLine(builder.GetInsertCommand().CommandText);
// INSERT INTO [Users] ([Name], [Age]) VALUES (@p1, @p2)

Console.WriteLine(builder.GetUpdateCommand().CommandText);
// UPDATE [Users] SET [Name] = @p1, [Age] = @p2 
// WHERE ([Id] = @p3 AND [Name] = @p4 AND [Age] = @p5)

Console.WriteLine(builder.GetDeleteCommand().CommandText);
// DELETE FROM [Users] WHERE ([Id] = @p6 AND [Name] = @p7 AND [Age] = @p8)
```

Notice the generated `UPDATE` and `DELETE` commands include ==all columns in the `WHERE` clause== — this is **optimistic concurrency** by default. The command only modifies a row if every column still matches the values originally read.

### Limitations

```ad-warning
title: CommandBuilder Limitations
`DbCommandBuilder` only works under these conditions:

1. **Single table** — the `SelectCommand` must query exactly one table (no joins)
2. **Primary key or unique column** — the table must have a primary key or at least one unique column in the result set
3. **No computed columns** — columns with expressions are excluded from generated commands
4. **No stored procedures** — cannot generate commands for stored procedure calls
5. **No custom SQL** — the generated SQL may not match your specific needs (e.g., different concurrency strategies, returning generated values, triggers)
6. **Performance** — the builder queries schema metadata, adding overhead on first use

For anything beyond basic single-table CRUD, ==set the commands manually==.
```

### Refreshing Commands

If you change the `SelectCommand` after creating the builder, you must refresh:

```csharp
adapter.SelectCommand.CommandText = "SELECT Id, Name, Age, Email FROM Users";
builder.RefreshSchema(); // regenerates commands to include Email column
```

Without `RefreshSchema()`, the builder continues using commands generated from the original `SelectCommand`.

```ad-note
title: Section Summary
- `CommandBuilder` auto-generates `INSERT`/`UPDATE`/`DELETE` from the `SelectCommand`
- Generated commands use optimistic concurrency (all columns in `WHERE` clause)
- Only works for single-table queries with a primary key — no joins, stored procs, or custom SQL
- Call `RefreshSchema()` if the `SelectCommand` changes after builder creation
```

---

## Batch Updating

By default, `Update()` sends one SQL command per row. For large batches, this is slow. **Batch updating** sends multiple commands in a single round trip:

```csharp
// Default: one command per round trip
adapter.UpdateBatchSize = 1;  // default

// Batch: send 100 commands per round trip
adapter.UpdateBatchSize = 100;

// Maximum batch: send all commands in one round trip
adapter.UpdateBatchSize = 0;  // 0 = no limit

adapter.Update(table);
```

| `UpdateBatchSize` | Behavior |
|---|---|
| `1` (default) | One command per database round trip |
| `n > 1` | Up to `n` commands per round trip |
| `0` | All commands in a single round trip |

```ad-warning
title: Provider Support Varies
Not all providers support batch updating. `SqlDataAdapter` (SQL Server) fully supports it. Check your provider's documentation. If the provider doesn't support batching, setting `UpdateBatchSize` has no effect — it silently falls back to one-at-a-time.
```

```ad-info
title: Performance Tip
For inserting thousands of rows, even batched `Update()` is slower than provider-specific bulk operations like `SqlBulkCopy` (SQL Server) or `LOAD DATA INFILE` (MySQL). Use `Update()` for moderate-size change sets (hundreds to low thousands of rows) where change tracking and mixed `INSERT`/`UPDATE`/`DELETE` are needed.
```

```ad-note
title: Section Summary
- Set `UpdateBatchSize` to send multiple commands per database round trip
- `0` means unlimited batch size; `1` (default) means one command per round trip
- Provider support varies — SQL Server supports batching; others may not
- For bulk inserts (10,000+ rows), consider `SqlBulkCopy` instead
```

---

## Table Mappings

When a `SelectCommand` returns results, the adapter uses **table mappings** to match result-set columns and table names to `DataTable` column and table names:

```csharp
// Rename the result table
adapter.TableMappings.Add("Table", "Users");
// "Table" is the default name for the first result set
// "Table1" for the second, "Table2" for the third, etc.

// Rename columns
var mapping = adapter.TableMappings.Add("Table", "Users");
mapping.ColumnMappings.Add("user_id", "Id");
mapping.ColumnMappings.Add("user_name", "Name");
mapping.ColumnMappings.Add("user_age", "Age");
// Now the DataTable columns are named "Id", "Name", "Age" 
// instead of "user_id", "user_name", "user_age"

adapter.Fill(ds);
DataTable users = ds.Tables["Users"]!;
Console.WriteLine(users.Columns[0].ColumnName); // "Id" (mapped from "user_id")
```

The `MissingMappingAction` property controls what happens when a column in the result set has no mapping:

| Value | Behavior |
|---|---|
| `Passthrough` (default) | Use the source column name as-is |
| `Ignore` | Skip unmapped columns |
| `Error` | Throw a `SystemException` |

The `MissingSchemaAction` property controls what happens when a column in the result set doesn't exist in the `DataTable`:

| Value | Behavior |
|---|---|
| `Add` (default) | Add the column to the table |
| `AddWithKey` | Add the column with primary key and unique constraint info |
| `Ignore` | Skip the column |
| `Error` | Throw a `SystemException` |

```ad-note
title: Section Summary
- Table mappings rename result-set tables and columns to `DataTable`/`DataColumn` names
- Default result-set names are "Table", "Table1", "Table2" — map them to meaningful names
- `MissingMappingAction` and `MissingSchemaAction` control behavior for unmapped/unknown columns
```

---

## Handling Events — RowUpdating and RowUpdated

The adapter fires events before and after each row is sent to the database, giving you fine-grained control:

```csharp
adapter.RowUpdating += (sender, e) =>
{
    // Fires BEFORE the command is executed for this row
    Console.WriteLine($"About to {e.StatementType} row: {e.Row["Name"]}");
    
    // You can modify the command, skip the row, or abort
    if ((string)e.Row["Name"] == "SKIP_ME")
    {
        e.Status = UpdateStatus.SkipCurrentRow;
    }
};

adapter.RowUpdated += (sender, e) =>
{
    // Fires AFTER the command is executed for this row
    Console.WriteLine($"{e.StatementType} affected {e.RecordsAffected} rows");
    
    // Handle concurrency violations
    if (e.RecordsAffected == 0)
    {
        Console.WriteLine($"Concurrency conflict on row {e.Row["Id"]}!");
        e.Status = UpdateStatus.SkipCurrentRow; // don't throw, just skip
    }
};
```

**`UpdateStatus` values:**

| Value | Behavior |
|---|---|
| `Continue` (default) | Proceed normally |
| `SkipCurrentRow` | Skip this row, continue with the next |
| `SkipAllRemainingRows` | Stop processing — no more rows are updated |
| `ErrorsOccurred` | Throw an exception (used with `ContinueUpdateOnError`) |

**`ContinueUpdateOnError`:**

```csharp
// Default: false — first error throws an exception and stops Update()
adapter.ContinueUpdateOnError = true;

adapter.Update(table);

// Check which rows had errors
foreach (DataRow row in table.GetErrors())
{
    Console.WriteLine($"Row {row["Id"]}: {row.RowError}");
}
```

When `ContinueUpdateOnError = true`, failed rows retain their `RowState` (not reset to `Unchanged`) and have `RowError` set with the error message. Successfully updated rows have `AcceptChanges()` called normally.

```ad-note
title: Section Summary
- `RowUpdating` fires before each command; `RowUpdated` fires after — both allow skipping or aborting
- `ContinueUpdateOnError = true` allows `Update()` to proceed past failures; check `table.GetErrors()` afterwards
- Failed rows keep their `RowState` and have `RowError` set; successful rows get `AcceptChanges()` called
```

---

## Concurrency Conflicts

When multiple users modify the same data, **optimistic concurrency** detects conflicts during `Update()`:

```csharp
// The CommandBuilder generates WHERE clauses with ALL original values:
// UPDATE Users SET Name = @new WHERE Id = @origId AND Name = @origName AND Age = @origAge

// If another user changed Name between your Fill() and Update(),
// the WHERE clause won't match → 0 rows affected → DBConcurrencyException
```

**Handling the exception:**

```csharp
try
{
    adapter.Update(table);
}
catch (DBConcurrencyException ex)
{
    DataRow conflictRow = ex.Row!;
    Console.WriteLine($"Conflict on row: {conflictRow["Id", DataRowVersion.Original]}");
    
    // Strategy 1: "Last write wins" — re-fill and try again
    adapter.Fill(table);
    conflictRow["Name"] = "My Value";  // re-apply your change
    adapter.Update(table);
    
    // Strategy 2: Show both versions to the user
    // ex.Row has Current (your changes) and Original (what you read)
    // Re-fill to get the other user's version, then let the user decide
    
    // Strategy 3: Merge changes
    DataTable freshData = new DataTable();
    adapter.Fill(freshData);
    table.Merge(freshData, preserveChanges: true);
    // preserveChanges: true keeps your modifications but updates Original versions
    adapter.Update(table);
}
```

**Concurrency strategies:**

| Strategy | WHERE Clause | Trade-off |
|---|---|---|
| **All columns** (CommandBuilder default) | `WHERE Id = @orig AND Name = @origN AND Age = @origA` | Detects any change; most restrictive |
| **Primary key + timestamp** | `WHERE Id = @orig AND RowVersion = @origTS` | Detects changes efficiently with a `rowversion`/`timestamp` column |
| **Primary key only** ("last write wins") | `WHERE Id = @orig` | No conflict detection; last update overwrites all |

```ad-info
title: Using a Timestamp / RowVersion Column
The most efficient optimistic concurrency approach is a `rowversion` (SQL Server) or equivalent column. The database auto-increments this value on every update. Your `WHERE` clause only needs `Id = @id AND RowVer = @origVer` instead of checking every column. Include the `rowversion` column in your `SELECT` and use it in manual `UPDATE`/`DELETE` commands.
```

```ad-note
title: Section Summary
- Optimistic concurrency detects conflicts by including original values in the `WHERE` clause
- `DBConcurrencyException` is thrown when `Update()` affects 0 rows (another user changed the data)
- Common strategies: all-columns check (CommandBuilder default), timestamp-based, or last-write-wins
- Use `DataTable.Merge()` with `preserveChanges: true` to combine fresh data with local modifications
```

---

## Async Operations

In modern .NET, `DbDataAdapter` has **limited async support**. The `Fill()` and `Update()` methods are ==synchronous only== in the base class:

```csharp
// Fill and Update are synchronous
adapter.Fill(table);    // blocks the thread
adapter.Update(table);  // blocks the thread
```

```ad-warning
title: No Built-in FillAsync or UpdateAsync
Unlike `DbConnection` and `DbCommand` which have full `Async` variants (`OpenAsync`, `ExecuteReaderAsync`, etc.), `DbDataAdapter` does ==not have `FillAsync()` or `UpdateAsync()`== in the base class. This is a known gap in the ADO.NET API.

**Workarounds:**
1. **Offload to a thread pool thread** — `await Task.Run(() => adapter.Fill(table))` (acceptable for desktop/WinForms apps; avoid in ASP.NET Core where thread pool threads are precious)
2. **Use `DbDataReader` directly** — for async reads, skip the adapter and use `await cmd.ExecuteReaderAsync()` to populate a `DataTable` manually
3. **Some providers may add async adapter methods** — check your specific provider's documentation
```

Manual async fill using `DbDataReader`:

```csharp
public static async Task<DataTable> FillAsync(DbCommand command)
{
    var table = new DataTable();
    
    await using var reader = await command.ExecuteReaderAsync();
    table.Load(reader); // DataTable.Load() accepts a DbDataReader
    
    return table;
}

// Usage
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();
using var cmd = new SqlCommand("SELECT Id, Name, Age FROM Users", conn);

DataTable table = await FillAsync(cmd);
```

```ad-info
title: DataTable.Load(IDataReader)
`DataTable.Load()` accepts any `IDataReader` (including `DbDataReader`) and populates the table from it. This is how `Fill()` works internally. The `Load()` method itself is synchronous, but the reader can be obtained asynchronously. This gives you async connection and query execution even though the final load step is synchronous.
```

```ad-note
title: Section Summary
- `DbDataAdapter.Fill()` and `Update()` are synchronous — no built-in async variants exist
- For async reads, use `ExecuteReaderAsync()` + `DataTable.Load(reader)` as a workaround
- `Task.Run(() => adapter.Fill(table))` is acceptable in desktop apps but not recommended in ASP.NET Core
```

---

## Comprehensive Summary

```ad-important
title: Key Takeaways
**DbDataAdapter** is the bridge between the database and `DataSet`/`DataTable`. It has four commands: `SelectCommand` (for `Fill`), `InsertCommand`, `UpdateCommand`, and `DeleteCommand` (for `Update`).

**Fill()** executes the `SelectCommand`, populates a `DataTable`, sets all rows to `Unchanged`, and ==auto-manages the connection== (opens if closed, closes after). Repeated `Fill()` without `Clear()` or a primary key causes duplicate rows.

**Update()** iterates every row in the `DataTable` and executes the appropriate command based on `RowState`:
- `Added` → `InsertCommand`
- `Modified` → `UpdateCommand`
- `Deleted` → `DeleteCommand`
- `Unchanged` / `Detached` → skipped

After successful execution, `AcceptChanges()` is called on each row automatically.

**DbCommandBuilder** auto-generates `INSERT`/`UPDATE`/`DELETE` commands from the `SelectCommand`, but ==only works for single-table queries with a primary key==. For joins, stored procedures, or custom concurrency strategies, set the commands manually.

**Parameter mapping** connects command parameters to `DataRow` columns via `SourceColumn` and `SourceVersion`. Use `SourceVersion = Original` for `WHERE` clause parameters to support optimistic concurrency.

**Optimistic concurrency** is built into the `CommandBuilder` — generated commands include all original column values in the `WHERE` clause. If another user changed the row, 0 rows are affected and a `DBConcurrencyException` is thrown.

**Key gap**: `Fill()` and `Update()` are synchronous. For async data access, use `ExecuteReaderAsync()` + `DataTable.Load()` or prefer the connected layer with `DbDataReader`.
```

---

## Related Topics

- [[DataSet and DataTable]] — the in-memory data structures that `DataAdapter` fills and updates
- [[DataView]] — sorting and filtering `DataTable` data in memory
- [[Connected vs Disconnected Layer]] — comparing the two ADO.NET approaches
- [[ADO.NET Overview]] — the overall ADO.NET architecture and provider model
- [[DbCommand]] — the command objects used by the adapter's four command properties
- [[Parameters and SQL Injection]] — parameterized queries and `SourceColumn` / `SourceVersion`
- [[Transactions in ADO.NET]] — wrapping `Update()` in a transaction for atomicity
- [[DbDataReader]] — the connected-layer alternative (and what `Fill()` uses internally)
