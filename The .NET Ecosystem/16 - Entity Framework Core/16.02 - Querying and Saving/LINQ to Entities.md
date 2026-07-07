---
tags: [csharp, ef-core, linq, querying]
---

- When you write LINQ against a `DbSet<T>`, EF Core **translates your C# expression tree into SQL** and sends it to the database. You write C# — the database sees SQL. Understanding this translation is the single most important skill for using EF Core effectively.

---

## How the Translation Works

- `DbSet<T>` implements `IQueryable<T>`, not just `IEnumerable<T>`.
- `IQueryable<T>` stores your LINQ calls as an **expression tree** (data structure describing the query) rather than executing them in memory.
- When you trigger execution (see below), EF Core's **query pipeline** walks the expression tree, converts it to SQL, sends it to the database, and maps the result rows back to C# objects.

```csharp
// C# LINQ
var redCars = db.Cars
    .Where(c => c.Color == "Red")
    .ToList();

// EF Core translates this to:
// SELECT [c].[Id], [c].[Make], [c].[Model], [c].[Color]
// FROM [Cars] AS [c]
// WHERE [c].[Color] = N'Red'
```

```ad-warning
title: IQueryable vs IEnumerable — the critical difference
If you accidentally cast to `IEnumerable<T>` (e.g., by calling a method EF can't translate), everything **after** that point runs in memory, not in SQL. This means the database sends all rows and C# filters them — terrible for performance.

```csharp
// BAD: MyCustomMethod can't be translated to SQL
// EF loads ALL cars, then filters in memory
var result = db.Cars
    .AsEnumerable()               // switches to LINQ-to-Objects
    .Where(c => MyCustomMethod(c)) // runs in C# memory
    .ToList();
```                                                     
```

---

## Deferred vs Immediate Execution

This works exactly like [[Deferred Execution in LINQ|LINQ-to-Objects deferred execution]], but the stakes are higher because a database round-trip is involved.

**Deferred** — the query is *built* but not yet sent to the database:
```csharp
// No SQL executed yet — just building an expression tree
var query = db.Cars
    .Where(c => c.Year > 2020)
    .OrderBy(c => c.Make);
```

**Immediate** — these methods trigger SQL execution:

| Trigger method | What it does |
|---|---|
| `ToList()` / `ToArray()` | Materializes all results into a collection |
| `FirstOrDefault()` / `First()` | Returns the first row (adds `TOP 1`) |
| `SingleOrDefault()` / `Single()` | Returns one row, throws if more than one |
| `Count()` / `LongCount()` | Returns `COUNT(*)` |
| `Any()` | Returns `EXISTS(...)` — `true`/`false` |
| `Sum()` / `Average()` / `Min()` / `Max()` | Aggregate functions |
| `foreach` loop | Iterates by streaming rows from DB |

```ad-tip
title: Chaining builds the query, terminal methods execute it
Think of it as two phases: **build** (chaining `Where`, `OrderBy`, `Select`, etc.) and **execute** (calling `ToList`, `First`, `Count`, etc.). You can chain as many operations as you want before executing — EF Core combines them all into one SQL statement.
```

---

## Common LINQ Methods and Their SQL Equivalents

| LINQ Method | SQL Equivalent | Example |
|---|---|---|
| `Where(predicate)` | `WHERE` | `.Where(c => c.Color == "Red")` |
| `Select(projection)` | `SELECT columns` | `.Select(c => new { c.Make, c.Model })` |
| `OrderBy()` / `OrderByDescending()` | `ORDER BY` / `ORDER BY ... DESC` | `.OrderBy(c => c.Year)` |
| `ThenBy()` / `ThenByDescending()` | Secondary `ORDER BY` column | `.OrderBy(c => c.Make).ThenBy(c => c.Model)` |
| `Include()` | `LEFT JOIN` | `.Include(c => c.Owner)` |
| `GroupBy(key)` | `GROUP BY` | `.GroupBy(c => c.Make)` |
| `Any(predicate)` | `EXISTS (SELECT 1 ... WHERE)` | `.Any(c => c.Year > 2020)` |
| `Count()` | `COUNT(*)` | `.Count(c => c.Color == "Red")` |
| `First()` / `FirstOrDefault()` | `TOP 1` / `LIMIT 1` | `.FirstOrDefault(c => c.Id == 5)` |
| `Single()` / `SingleOrDefault()` | `TOP 2` (to detect duplicates) | `.SingleOrDefault(c => c.Vin == vin)` |
| `Skip(n).Take(m)` | `OFFSET n ROWS FETCH NEXT m` | `.Skip(20).Take(10)` — page 3 |
| `Distinct()` | `DISTINCT` | `.Select(c => c.Color).Distinct()` |
| `Join()` | `INNER JOIN` | Explicit join syntax |
| `Sum()` / `Average()` / `Min()` / `Max()` | `SUM` / `AVG` / `MIN` / `MAX` | `.Sum(c => c.Price)` |

---

## FirstOrDefault vs SingleOrDefault vs Find

These three are commonly confused. They have **different SQL and different intent**:

| Method | SQL generated | Throws if multiple? | Throws if none? | Checks cache? |
|---|---|---|---|---|
| `FirstOrDefault()` | `TOP 1` | No (returns first) | No (returns `null`) | No |
| `SingleOrDefault()` | `TOP 2` | Yes (`InvalidOperationException`) | No (returns `null`) | No |
| `Find(key)` | `SELECT ... WHERE Id = @p0` | N/A (by PK) | No (returns `null`) | **Yes** |

```ad-note
title: Find() checks the change tracker first
`Find()` is a method on `DbSet<T>`, not a LINQ operator. It looks up by **primary key** and checks the in-memory change tracker before querying the database. If the entity is already tracked (e.g., you loaded it earlier in the same `DbContext` lifetime), `Find()` returns it instantly with zero SQL. This makes it ideal for PK lookups. See [[Change Tracking]].
```

**When to use which:**
- `FirstOrDefault()` — "Give me any matching row, I don't care if there are more."
- `SingleOrDefault()` — "There should be exactly 0 or 1 — throw if something is wrong." Use for unique constraints, lookups by unique columns.
- `Find(key)` — "I have the primary key." Fastest option for PK lookups thanks to cache.

For more on the distinction between `Single` and `First`, see [[Single vs First]].

---

## IQueryable Chaining — Building Queries Dynamically

One of the most powerful patterns is building queries conditionally:

```csharp
// Start with a base query
IQueryable<Car> query = db.Cars;

// Conditionally add filters
if (!string.IsNullOrEmpty(colorFilter))
    query = query.Where(c => c.Color == colorFilter);

if (minYear.HasValue)
    query = query.Where(c => c.Year >= minYear.Value);

if (sortByPrice)
    query = query.OrderBy(c => c.Price);

// Only one SQL statement is generated, combining all applied filters
var results = await query.ToListAsync();
```

This pattern is used everywhere in real applications — search pages, report builders, API endpoints with optional filters. EF Core merges all the chained expressions into a single SQL query.

```ad-tip
title: Always use the async versions in web apps
EF Core provides async counterparts for every terminal method: `ToListAsync()`, `FirstOrDefaultAsync()`, `CountAsync()`, `AnyAsync()`, etc. In ASP.NET Core, always prefer async — it frees the thread while waiting for the database, allowing your server to handle more requests.
```

---

## Select Projections — Don't Load What You Don't Need

When you only need a few columns, use `Select()` to project into an anonymous type or DTO. EF Core will only `SELECT` those columns:

```csharp
// Only fetches Make, Model, and Year — not the entire Car entity
var carSummaries = db.Cars
    .Where(c => c.Year > 2020)
    .Select(c => new
    {
        c.Make,
        c.Model,
        c.Year
    })
    .ToList();

// SQL: SELECT [c].[Make], [c].[Model], [c].[Year]
//      FROM [Cars] AS [c]
//      WHERE [c].[Year] > 2020
```

```ad-note
title: Projected results are not tracked
When you use `Select()` to project into an anonymous type or a DTO (not an entity), the results are **not tracked** by the change tracker. This gives you the same performance benefit as `AsNoTracking()` automatically. See [[Change Tracking]].
```

---

## What LINQ Can't Translate

Not every C# expression can become SQL. When EF Core encounters something it can't translate, it either:
1. **Throws a runtime exception** (default in EF Core 3.0+) — this is good, it tells you about the problem.
2. Falls back to **client evaluation** (only for the final `Select` projection in specific cases).

Common things that **cannot** be translated:
- Custom C# methods: `.Where(c => MyHelperMethod(c.Name))` 
- Most `string` methods beyond `Contains`, `StartsWith`, `EndsWith`, `ToUpper`, `ToLower`
- Constructor calls in `Where` clauses
- Complex object comparisons

```ad-warning
title: "LINQ expression could not be translated" error
If you see this exception, you have two options:
1. **Rewrite** the query using translatable expressions (preferred).
2. **Split** the query: run the translatable part in SQL, then call `.AsEnumerable()` and run the rest in memory. Be careful about how many rows you pull from the database.
```

---

## See Also

- [[LINQ Introduction]] — the fundamentals of LINQ syntax and concepts
- [[Deferred Execution in LINQ]] — how deferred execution works in LINQ-to-Objects
- [[Immediate Execution in LINQ]] — methods that force immediate execution
- [[Change Tracking]] — how EF Core tracks query results
- [[Loading Related Data]] — Include/ThenInclude for navigation properties
- [[Single vs First]] — detailed comparison of these operators
