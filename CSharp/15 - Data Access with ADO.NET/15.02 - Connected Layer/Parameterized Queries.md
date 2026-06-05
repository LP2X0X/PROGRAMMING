---
tags:
 - csharp
 - ado-net
 - security
---

## Parameterized Queries -- The Only Safe Way to Pass User Input to SQL

Parameterized queries are the **mandatory** technique for incorporating dynamic values (especially user input) into SQL statements. Instead of concatenating values directly into the SQL string, you use **placeholders** (parameters) in the SQL text and bind values separately. The database engine treats the parameter values strictly as **data**, never as executable **code**, which eliminates SQL injection entirely.

---

## SQL Injection -- Why String Concatenation Is Dangerous

SQL injection is consistently ranked as the **#1 database security vulnerability** (OWASP Top 10). It occurs when user-supplied input is embedded directly into a SQL string without proper handling.

### The Vulnerability

```csharp
// VULNERABLE -- never do this
string userInput = GetUserInput(); // could be anything
string sql = $"SELECT * FROM Users WHERE Name = '{userInput}'";
using var cmd = new SqlCommand(sql, conn);
using var reader = cmd.ExecuteReader();
```

If the user enters normal text like `Long`, the query is:
```sql
SELECT * FROM Users WHERE Name = 'Long'
```

But if the user enters `'; DROP TABLE Users; --`, the query becomes:
```sql
SELECT * FROM Users WHERE Name = ''; DROP TABLE Users; --'
```

The database executes **three** statements:
1. `SELECT * FROM Users WHERE Name = ''` -- harmless
2. `DROP TABLE Users` -- **catastrophic**
3. `--'` -- a comment that neutralizes the trailing quote

```ad-warning
title: SQL Injection Is Not Theoretical
SQL injection is routinely exploited in the real world. Even "internal only" applications are at risk -- malicious actors inside a network, or any future exposure to the internet, can exploit unparameterized queries. ==There is never a valid reason to concatenate user input into SQL.==
```

### Other Injection Variants

| Attack | Injected Input | Effect |
|---|---|---|
| Data exfiltration | `' UNION SELECT Password FROM AdminUsers --` | Returns sensitive data from other tables |
| Authentication bypass | `' OR 1=1 --` | Makes `WHERE` clause always true |
| Blind injection | `' AND (SELECT COUNT(*) FROM Users) > 0 --` | Extracts data via true/false responses |
| Time-based blind | `'; WAITFOR DELAY '00:00:05' --` | Confirms vulnerability via response timing |

---

## The Solution -- Parameterized Queries

```csharp
// SAFE -- parameterized query
using var cmd = new SqlCommand("SELECT * FROM Users WHERE Name = @Name", conn);
cmd.Parameters.AddWithValue("@Name", userInput);
using var reader = cmd.ExecuteReader();
```

No matter what `userInput` contains -- even `'; DROP TABLE Users; --` -- the database treats the entire value as a **literal string** to compare against the `Name` column. The SQL text and the parameter values are sent to the database **separately**. The database compiles the SQL first, then plugs in the parameter values as data.

### How It Works Under the Hood

The flow for a parameterized query:

1. **Application sends**: SQL text with placeholders + parameter values as separate protocol elements
2. **Database receives**: Parses and compiles the SQL text into an execution plan (with placeholders)
3. **Database binds**: Substitutes parameter values into the compiled plan as **typed data values**
4. **Database executes**: Runs the plan with the bound values

Because the parameter values are never part of the SQL parsing step, they cannot alter the structure of the query. This is fundamentally different from string concatenation, where the database receives one combined string and cannot distinguish SQL from data.

```ad-note
title: Execution Plan Caching Benefit
Parameterized queries also improve performance. Because the SQL text is the same every time (only the parameter values change), the database can **cache and reuse the execution plan**. With string concatenation, every unique input generates a different SQL string, and the database must compile a new plan each time.
```

---

## Adding Parameters to DbCommand

### Method 1: AddWithValue() -- Convenient but Type-Inferred

```csharp
cmd.Parameters.AddWithValue("@Name", "Long");
cmd.Parameters.AddWithValue("@Age", 30);
cmd.Parameters.AddWithValue("@IsActive", true);
cmd.Parameters.AddWithValue("@JoinDate", DateTime.Now);
```

`AddWithValue()` infers the `SqlDbType` from the .NET type of the value you pass:

| .NET Type | Inferred SqlDbType |
|---|---|
| `string` | `NVarChar` |
| `int` | `Int` |
| `long` | `BigInt` |
| `bool` | `Bit` |
| `DateTime` | `DateTime` |
| `decimal` | `Decimal` |
| `double` | `Float` |
| `Guid` | `UniqueIdentifier` |
| `byte[]` | `VarBinary` |

```ad-warning
title: The AddWithValue Problem -- Implicit Type Conversion
`AddWithValue()` infers `NVarChar` for strings, but if your column is `VARCHAR`, the database must implicitly convert every row's value from `VARCHAR` to `NVARCHAR` to compare. This **prevents index usage** and forces a full table scan. This is known as an **implicit conversion performance problem** and is one of the most common ADO.NET performance traps.

Example: if column `Email` is `VARCHAR(100)` and you pass `AddWithValue("@Email", "test@example.com")`, the parameter becomes `NVARCHAR` and the database converts every row's `Email` to `NVARCHAR` for comparison -- even if there's an index on `Email`.
```

### Method 2: Add() with Explicit Type -- Preferred for Production Code

```csharp
// Explicit type and size -- no implicit conversion risk
cmd.Parameters.Add("@Name", SqlDbType.NVarChar, 100).Value = "Long";
cmd.Parameters.Add("@Age", SqlDbType.Int).Value = 30;
cmd.Parameters.Add("@Email", SqlDbType.VarChar, 200).Value = "long@example.com";
cmd.Parameters.Add("@Salary", SqlDbType.Decimal).Value = 75000.00m;
cmd.Parameters.Add("@JoinDate", SqlDbType.DateTime2).Value = DateTime.Now;
```

### Method 3: Constructing a DbParameter Object

```csharp
var param = new SqlParameter
{
    ParameterName = "@Name",
    SqlDbType     = SqlDbType.NVarChar,
    Size          = 100,
    Value         = "Long"
};
cmd.Parameters.Add(param);

// Shorthand constructor
var param2 = new SqlParameter("@Id", SqlDbType.Int) { Value = 42 };
cmd.Parameters.Add(param2);
```

### Comparison of Methods

| Method | Type Safety | Performance | Convenience | Recommended For |
|---|---|---|---|---|
| `AddWithValue()` | Inferred (risky) | Possible implicit conversions | Highest | Quick prototypes, scripts |
| `Add(name, type, size)` | Explicit | Optimal | Moderate | Production code |
| `new SqlParameter(...)` | Explicit | Optimal | Lowest | Complex scenarios (output, precision) |

---

## Passing NULL Values

To pass SQL `NULL` as a parameter value, use `DBNull.Value` -- not C# `null`.

```csharp
// WRONG -- C# null causes an error or sends no value
cmd.Parameters.AddWithValue("@MiddleName", null);  // throws or sends empty

// CORRECT -- use DBNull.Value for SQL NULL
cmd.Parameters.AddWithValue("@MiddleName", DBNull.Value);

// Common pattern: coalesce C# null to DBNull.Value
string? middleName = GetMiddleName();
cmd.Parameters.AddWithValue("@MiddleName", (object?)middleName ?? DBNull.Value);

// With explicit type
cmd.Parameters.Add("@MiddleName", SqlDbType.NVarChar, 50).Value =
    (object?)middleName ?? DBNull.Value;
```

```ad-warning
title: Passing C# null as a Parameter Value
If you pass C# `null` (not `DBNull.Value`) to `AddWithValue()`, the behavior is provider-specific. `SqlCommand` interprets it as "do not send a value for this parameter," which causes the stored procedure to use its default value (if one exists) or throws an error. ==Always use `DBNull.Value` to represent SQL NULL.==
```

---

## Output Parameters

Parameters can flow in both directions. **Output parameters** let the database return values back to the caller, commonly used with [[Stored Procedures]].

### ParameterDirection Enum

| Direction | Description |
|---|---|
| `Input` (default) | Value is sent from application to database |
| `Output` | Value is returned from database to application (initial value ignored) |
| `InputOutput` | Value is sent to the database and may be modified and returned |
| `ReturnValue` | Captures the stored procedure's `RETURN` value |

### Output Parameter Example

```csharp
cmd.CommandType = CommandType.StoredProcedure;
cmd.CommandText = "sp_GetUserCount";

var outputParam = new SqlParameter("@Count", SqlDbType.Int)
{
    Direction = ParameterDirection.Output
};
cmd.Parameters.Add(outputParam);

cmd.ExecuteNonQuery();

int count = (int)outputParam.Value;
Console.WriteLine($"User count: {count}");
```

### InputOutput Parameter Example

```csharp
// The stored procedure reads the input and modifies it
var param = new SqlParameter("@Balance", SqlDbType.Decimal)
{
    Direction = ParameterDirection.InputOutput,
    Value     = 500.00m  // initial balance sent to the proc
};
cmd.Parameters.Add(param);
cmd.ExecuteNonQuery();

decimal newBalance = (decimal)param.Value;  // modified value returned
```

### ReturnValue Parameter

```csharp
var returnParam = new SqlParameter
{
    Direction = ParameterDirection.ReturnValue,
    SqlDbType = SqlDbType.Int
};
cmd.Parameters.Add(returnParam);
cmd.ExecuteNonQuery();

int returnCode = (int)returnParam.Value;
// Convention: 0 = success, non-zero = error
```

```ad-note
title: Output Parameters Are Available After the Reader Is Closed
When using `ExecuteReader()` with output parameters, the output values are ==not available until the reader is closed==. You must call `reader.Close()` (or dispose it) before reading `outputParam.Value`. This is because the output values are returned as part of the TDS protocol stream after all result set rows.
```

---

## Parameter Placeholder Syntax by Provider

Different ADO.NET providers use different placeholder syntax:

| Provider | Prefix | Example |
|---|---|---|
| SQL Server (`SqlCommand`) | `@` | `WHERE Name = @Name` |
| MySQL (`MySqlCommand`) | `@` | `WHERE Name = @Name` |
| PostgreSQL (`NpgsqlCommand`) | `@` or `$` | `WHERE Name = @Name` or `WHERE Name = $1` |
| Oracle (`OracleCommand`) | `:` | `WHERE Name = :Name` |
| OLE DB (`OleDbCommand`) | `?` (positional) | `WHERE Name = ?` |
| ODBC (`OdbcCommand`) | `?` (positional) | `WHERE Name = ?` |

```ad-note
title: Positional Parameters (OLE DB / ODBC)
OLE DB and ODBC use `?` as a positional placeholder. Parameters must be added to the collection in the exact order they appear in the SQL. Named parameters are not supported -- the `ParameterName` is ignored; only the position in the collection matters.
```

---

## Parameterizing Non-Value Elements

Parameters can only replace **values** in SQL -- they cannot replace table names, column names, keywords, or other structural elements.

```csharp
// This does NOT work -- table names cannot be parameterized
cmd.CommandText = "SELECT * FROM @TableName";         // ERROR
cmd.Parameters.AddWithValue("@TableName", "Users");

// This does NOT work -- column names cannot be parameterized
cmd.CommandText = "SELECT * FROM Users ORDER BY @Column";  // ERROR
cmd.Parameters.AddWithValue("@Column", "Name");
```

If you must use dynamic table or column names, use a **whitelist** approach:

```csharp
// Safe: validate against a known list of allowed values
string[] allowedColumns = { "Name", "Age", "Email", "JoinDate" };
if (!allowedColumns.Contains(sortColumn))
    throw new ArgumentException($"Invalid sort column: {sortColumn}");

cmd.CommandText = $"SELECT * FROM Users ORDER BY {sortColumn}";
// sortColumn is from the whitelist -- safe to interpolate
```

```ad-warning
title: Whitelisting Is the Only Safe Alternative
When you must use dynamic identifiers, ==never rely on escaping or quoting==. Always validate against a fixed whitelist of known-good values. Any other approach is fragile and potentially exploitable.
```

---

## Summary

| Concept | Detail |
|---|---|
| What they are | SQL queries where dynamic values are passed as separate parameters, not concatenated into the SQL string |
| Why | Prevents SQL injection; enables execution plan reuse |
| `AddWithValue()` | Convenient but infers `SqlDbType` -- can cause implicit conversion performance issues |
| `Add(name, type, size)` | Explicit type -- preferred for production code |
| NULL values | Use `DBNull.Value`, not C# `null` |
| `ParameterDirection` | `Input` (default), `Output`, `InputOutput`, `ReturnValue` |
| Placeholder syntax | `@Param` (SQL Server, MySQL, PostgreSQL), `:Param` (Oracle), `?` (OLE DB, ODBC) |
| Limitations | Parameters replace **values only** -- not table names, column names, or SQL keywords |
| Dynamic identifiers | Use a whitelist of allowed values; never escape or quote user input |
