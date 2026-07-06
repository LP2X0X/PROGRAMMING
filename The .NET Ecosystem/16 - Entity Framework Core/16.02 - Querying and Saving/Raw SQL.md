---
tags: [csharp, ef-core, raw-sql, performance]
---

- EF Core can translate most queries, but sometimes you need raw SQL: complex queries that LINQ can't express, performance-critical bulk operations, database-specific features (CTEs, window functions, full-text search), or calling stored procedures. EF Core provides several methods to execute raw SQL while still giving you parameterization, tracking, and type mapping.

---

## When to Use Raw SQL

| Scenario | Reason |
|---|---|
| Complex queries with CTEs, recursive queries, window functions | LINQ can't express these |
| Performance-critical bulk operations | Per-row EF approach is too slow |
| Database-specific features (full-text search, JSON operators) | Not translatable by EF Core's LINQ provider |
| Calling [[Stored Procedures]] | Need to invoke existing DB logic |
| Existing SQL you've already optimized | Don't want to rewrite in LINQ |
| Queries that EF Core translates poorly | Check the generated SQL — sometimes hand-written SQL is better |

---

## FromSqlRaw — Returning Entities

Returns entities that are **tracked** by the change tracker (just like a normal LINQ query):

```csharp
// Basic usage with parameter placeholder
var cars = db.Cars
    .FromSqlRaw("SELECT * FROM Cars WHERE Color = {0}", color)
    .ToList();

// The result is List<Car> — fully tracked entities
// You can chain LINQ on top:
var sortedCars = db.Cars
    .FromSqlRaw("SELECT * FROM Cars WHERE Year > {0}", 2020)
    .OrderBy(c => c.Make)          // added to the SQL as ORDER BY
    .Where(c => c.Color == "Red")  // added as AND Color = 'Red'
    .ToList();
```

```ad-warning
title: FromSqlRaw must return ALL columns of the entity
The SQL must select every column that the entity type maps to. If your `Car` entity has `Id`, `Make`, `Model`, `Year`, `Color`, your SQL must return all five. You can't do `SELECT Make, Model FROM Cars` — EF will throw because it can't fully materialize the entity.

If you only need some columns, use `FromSqlRaw` + `.Select()` or use `SqlQueryRaw<T>` for non-entity types.
```

### Parameter Placeholders

`FromSqlRaw` uses **positional placeholders** (`{0}`, `{1}`), not string interpolation. EF Core converts these to proper SQL parameters:

```csharp
// CORRECT: positional placeholders — parameterized, safe
var cars = db.Cars
    .FromSqlRaw("SELECT * FROM Cars WHERE Make = {0} AND Year > {1}",
        make, year)
    .ToList();
// Sent to DB: SELECT * FROM Cars WHERE Make = @p0 AND Year > @p1
```

```ad-warning
title: Don't use string concatenation with FromSqlRaw
```csharp
// DANGEROUS: SQL injection vulnerability!
var cars = db.Cars
    .FromSqlRaw($"SELECT * FROM Cars WHERE Make = '{make}'")
    .ToList();
// If make = "'; DROP TABLE Cars; --" ... disaster
```
Always use the parameter placeholders. Or better yet, use `FromSqlInterpolated` (below).
```

---

## FromSqlInterpolated — Safe String Interpolation

The **preferred** method. Uses C# string interpolation but EF Core automatically extracts the interpolated values as SQL parameters:

```csharp
// SAFE: looks like interpolation, but EF parameterizes it
var cars = db.Cars
    .FromSqlInterpolated(
        $"SELECT * FROM Cars WHERE Make = {make} AND Year > {year}")
    .ToList();
// Sent to DB: SELECT * FROM Cars WHERE Make = @p0 AND Year > @p1
```

The difference from `FromSqlRaw`:
- `FromSqlRaw("...", param1, param2)` — string + separate params
- `FromSqlInterpolated($"... {param1} ... {param2}")` — interpolated FormattableString, automatically parameterized

```ad-tip
title: Prefer FromSqlInterpolated over FromSqlRaw
`FromSqlInterpolated` is harder to misuse because the parameterization is built into the syntax. With `FromSqlRaw`, you could accidentally use string interpolation instead of positional placeholders. See [[Parameterized Queries]] for why parameterization matters.
```

---

## ExecuteSqlRaw / ExecuteSqlInterpolated — Non-Query Commands

For `INSERT`, `UPDATE`, `DELETE` that **don't return entities**. Returns the number of affected rows:

```csharp
// ExecuteSqlRaw with positional placeholders
int rowsAffected = db.Database.ExecuteSqlRaw(
    "UPDATE Cars SET Color = {0} WHERE Year < {1}", "Classic Silver", 1980);

// ExecuteSqlInterpolated with safe interpolation
int rowsAffected = db.Database.ExecuteSqlInterpolated(
    $"DELETE FROM AuditLogs WHERE CreatedDate < {cutoffDate}");

Console.WriteLine($"{rowsAffected} rows affected");
```

```ad-warning
title: ExecuteSql bypasses the change tracker
These methods execute SQL directly on the database. Any entities currently tracked by the `DbContext` are **not updated** to reflect the changes. If you have a tracked Car entity and `ExecuteSqlRaw` changes its Color in the database, the tracked entity still has the old Color value. Either:
- Run `ExecuteSql` before loading entities, or
- Reload affected entities after: `db.Entry(car).Reload()`
```

---

## SqlQueryRaw / SqlQuery — Non-Entity Results (.NET 8+)

For queries that return **scalar values or custom types** (not mapped entities):

```csharp
// Return scalar values
var carNames = db.Database
    .SqlQueryRaw<string>("SELECT DISTINCT Make FROM Cars")
    .ToList();

// Return a custom DTO (not an entity)
var stats = db.Database
    .SqlQuery<CarStats>(
        $"SELECT Make, COUNT(*) AS Count, AVG(Year) AS AvgYear FROM Cars GROUP BY Make")
    .ToList();

// CarStats is a plain class — not a DbSet, not tracked
public class CarStats
{
    public string Make { get; set; }
    public int Count { get; set; }
    public double AvgYear { get; set; }
}
```

```ad-note
title: Before .NET 8
In earlier EF Core versions, returning non-entity types from raw SQL required either:
- Keyless entity types configured with `HasNoKey()` and added to the `DbContext` as `DbSet<T>`
- Dropping down to raw ADO.NET via `db.Database.GetDbConnection()`

`.SqlQueryRaw<T>` in .NET 8 greatly simplified this.
```

---

## Calling Stored Procedures

```csharp
// Stored procedure that returns entity-shaped results
var cars = db.Cars
    .FromSqlInterpolated($"EXEC GetCarsByMake {make}")
    .ToList();

// Stored procedure that doesn't return entities
db.Database.ExecuteSqlInterpolated(
    $"EXEC ArchiveOldOrders {cutoffDate}");

// Stored procedure with output parameter (requires ADO.NET)
var outputParam = new SqlParameter
{
    ParameterName = "@TotalCount",
    SqlDbType = System.Data.SqlDbType.Int,
    Direction = System.Data.ParameterDirection.Output
};

db.Database.ExecuteSqlRaw("EXEC CountCars @Make = {0}, @TotalCount = @TotalCount OUTPUT",
    make, outputParam);

int totalCount = (int)outputParam.Value;
```

For background on stored procedures, see [[Stored Procedures]].

---

## Composing LINQ on Top of Raw SQL

One powerful feature: you can chain LINQ methods after `FromSql`, and EF Core wraps your SQL as a subquery:

```csharp
var cars = db.Cars
    .FromSqlInterpolated($"SELECT * FROM Cars WHERE Year > {2020}")
    .Where(c => c.Color == "Red")    // added to SQL
    .OrderBy(c => c.Make)             // added to SQL
    .Take(10)                          // added to SQL
    .ToList();

// Approximate generated SQL:
// SELECT TOP(10) [c].*
// FROM (SELECT * FROM Cars WHERE Year > @p0) AS [c]
// WHERE [c].[Color] = N'Red'
// ORDER BY [c].[Make]
```

```ad-note
title: Composition requirements
For LINQ composition to work, your raw SQL must be composable — it can't contain features like `ORDER BY` (without `OFFSET`/`FETCH`), unions, or CTEs at the outermost level. EF Core needs to be able to wrap it as a subquery. If composition fails, EF throws at runtime.
```

---

## Summary Table — Which Method to Use

| Method | Returns | Tracked? | Use when |
|---|---|---|---|
| `FromSqlRaw` / `FromSqlInterpolated` | Entities (`DbSet<T>`) | Yes | Raw SQL that returns full entity rows |
| `ExecuteSqlRaw` / `ExecuteSqlInterpolated` | `int` (rows affected) | No | INSERT/UPDATE/DELETE, stored procs without results |
| `SqlQueryRaw<T>` / `SqlQuery<T>` (.NET 8+) | Custom type / scalar | No | Aggregates, DTOs, non-entity projections |
| ADO.NET (`GetDbConnection()`) | Whatever you want | No | Full control, complex output params, multiple result sets |

---

## See Also

- [[LINQ to Entities]] — when LINQ can handle the query, prefer it over raw SQL
- [[Stored Procedures]] — SQL stored procedure fundamentals
- [[Parameterized Queries]] — why parameterization prevents SQL injection
- [[Transactions]] — using raw SQL inside explicit transactions
- [[CRUD Operations]] — EF Core's standard create/update/delete patterns
