---
tags:
 - csharp
 - ado-net
 - database
---

## What Are Stored Procedures?

Stored procedures are precompiled SQL code stored in the database itself. Instead of sending raw SQL from your C# code, you call the procedure by name. The database compiles and caches the execution plan, so subsequent calls skip the compilation step.

```sql
-- SQL Server: creating a stored procedure
CREATE PROCEDURE sp_GetUserById
    @Id INT
AS
BEGIN
    SELECT Id, Name, Email, Age
    FROM Users
    WHERE Id = @Id
END
```


---

## Calling a Stored Procedure from C#

Set `CommandType` to `StoredProcedure` and pass the procedure name as `CommandText`:

```csharp
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();

using var cmd = new SqlCommand("sp_GetUserById", conn);
cmd.CommandType = CommandType.StoredProcedure;
cmd.Parameters.AddWithValue("@Id", 42);

using var reader = await cmd.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    Console.WriteLine($"{reader["Name"]} ({reader["Email"]})");
}
```

```ad-warning
If you forget to set `CommandType = CommandType.StoredProcedure`, ADO.NET treats the procedure name as raw SQL text and you'll get a syntax error.
```


---

## Input, Output, and Return Parameters

### Input Parameters (Default)

```csharp
cmd.Parameters.AddWithValue("@Name", "Long");
cmd.Parameters.AddWithValue("@Age", 28);
```

### Output Parameters

The stored procedure writes a value back to your code:

```sql
CREATE PROCEDURE sp_InsertUser
    @Name NVARCHAR(100),
    @Age INT,
    @NewId INT OUTPUT
AS
BEGIN
    INSERT INTO Users (Name, Age) VALUES (@Name, @Age)
    SET @NewId = SCOPE_IDENTITY()
END
```

```csharp
using var cmd = new SqlCommand("sp_InsertUser", conn);
cmd.CommandType = CommandType.StoredProcedure;

cmd.Parameters.AddWithValue("@Name", "Long");
cmd.Parameters.AddWithValue("@Age", 28);

var outputParam = new SqlParameter("@NewId", SqlDbType.Int)
{
    Direction = ParameterDirection.Output
};
cmd.Parameters.Add(outputParam);

await cmd.ExecuteNonQueryAsync();
int newId = (int)outputParam.Value;
Console.WriteLine($"Inserted with Id: {newId}");
```

### Return Value

Stored procedures can return an integer status code via `RETURN`:

```sql
CREATE PROCEDURE sp_CheckUserExists
    @Name NVARCHAR(100)
AS
BEGIN
    IF EXISTS (SELECT 1 FROM Users WHERE Name = @Name)
        RETURN 1
    RETURN 0
END
```

```csharp
using var cmd = new SqlCommand("sp_CheckUserExists", conn);
cmd.CommandType = CommandType.StoredProcedure;
cmd.Parameters.AddWithValue("@Name", "Long");

var returnParam = new SqlParameter
{
    Direction = ParameterDirection.ReturnValue
};
cmd.Parameters.Add(returnParam);

await cmd.ExecuteNonQueryAsync();
int exists = (int)returnParam.Value;
Console.WriteLine($"User exists: {exists == 1}");
```

### ParameterDirection Reference

| Direction | Meaning |
|---|---|
| `Input` (default) | Value sent to the procedure |
| `Output` | Value returned from the procedure |
| `InputOutput` | Value sent in, modified, returned |
| `ReturnValue` | Integer return code from `RETURN` |


---

## Stored Procedures That Return Result Sets

```csharp
// Single result set
using var cmd = new SqlCommand("sp_GetAllUsers", conn);
cmd.CommandType = CommandType.StoredProcedure;

using var reader = await cmd.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    Console.WriteLine(reader["Name"]);
}
```

```csharp
// Multiple result sets
using var cmd = new SqlCommand("sp_GetDashboardData", conn);
cmd.CommandType = CommandType.StoredProcedure;

using var reader = await cmd.ExecuteReaderAsync();

// First result set — Users
while (await reader.ReadAsync())
    Console.WriteLine($"User: {reader["Name"]}");

// Second result set — Orders
await reader.NextResultAsync();
while (await reader.ReadAsync())
    Console.WriteLine($"Order: {reader["OrderId"]}");
```


---

## Stored Procedures with Transactions

```csharp
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();
using var tx = conn.BeginTransaction();

try
{
    using var cmd = new SqlCommand("sp_TransferFunds", conn, tx);
    cmd.CommandType = CommandType.StoredProcedure;
    cmd.Parameters.AddWithValue("@FromAccount", 1);
    cmd.Parameters.AddWithValue("@ToAccount", 2);
    cmd.Parameters.AddWithValue("@Amount", 500.00m);

    await cmd.ExecuteNonQueryAsync();
    tx.Commit();
}
catch
{
    tx.Rollback();
    throw;
}
```


---

## Stored Procedures vs Inline SQL

| | Stored Procedures | Inline SQL (Parameterized) |
|---|---|---|
| SQL injection risk | None — parameters always safe | None if parameterized |
| Execution plan caching | Always cached by name | Cached per query text |
| Network traffic | Just procedure name + params | Full SQL text + params |
| Maintainability | SQL in database — DBA manages | SQL in C# code — developer manages |
| Deployment | Requires DB migration | Deploys with application |
| Flexibility | Fixed interface — must alter proc to change | Change SQL in code freely |
| Debugging | Harder — SQL is in the database | Easier — SQL is in your code |
| Version control | Requires separate SQL migration scripts | Naturally in source control |

```ad-note
Neither approach is universally better. Stored procedures are common in enterprise environments where DBAs control database access. Inline parameterized SQL is common in agile teams where developers own the full stack. Many modern apps use an ORM (EF Core) or micro-ORM (Dapper) instead of writing raw ADO.NET.
```


---

## Best Practices

1. **Always use `CommandType.StoredProcedure`** — don't call stored procedures as `EXEC sp_Name @param` in a text command
2. **Use explicit parameter types** — `Add("@Id", SqlDbType.Int)` instead of `AddWithValue` for better plan caching
3. **Handle output parameters after execution** — their `.Value` is only populated after `ExecuteNonQuery`/`ExecuteReader` completes
4. **Use `async` methods** — `ExecuteNonQueryAsync`, `ExecuteReaderAsync` for I/O-bound database calls


---

## See Also

- [[DbCommand]] — the command object that executes stored procedures
- [[Parameterized Queries]] — why parameters matter and how they work
- [[DbDataReader]] — reading result sets from stored procedures
- [[Transactions]] — wrapping stored procedure calls in transactions
