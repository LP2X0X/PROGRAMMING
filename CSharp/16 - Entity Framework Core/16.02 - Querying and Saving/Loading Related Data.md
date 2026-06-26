---
tags: [csharp, ef-core, querying, performance]
---

- EF Core does **not** load related entities automatically by default. If you load an `Order`, its `Order.Items` collection will be `null` or empty unless you explicitly tell EF Core to load it. This note covers the three strategies for loading related data and the critical **N+1 problem** you must avoid.

---

## The N+1 Problem

The N+1 problem is the most common performance mistake in any ORM. It happens when your code triggers one query per related entity instead of loading everything upfront.

```csharp
// BAD: N+1 — this generates 101 SQL queries!
var orders = db.Orders.ToList();            // 1 query: SELECT * FROM Orders (100 rows)

foreach (var order in orders)
{
    Console.WriteLine(order.Items.Count);    // 100 queries: one SELECT per order
    // Each access to order.Items triggers:
    // SELECT * FROM OrderItems WHERE OrderId = @id
}
```

**1 query** to load 100 orders + **100 queries** to load each order's items = **101 queries total**. This destroys performance, especially over a network.

```ad-warning
title: N+1 is silent and easy to miss
N+1 queries don't throw errors — your code works "correctly" but runs hundreds of times slower than it should. The fix is almost always eager loading (below). If you're seeing slow pages, check your SQL query count first.
```

---

## Strategy 1: Eager Loading (Include / ThenInclude)

**Eager loading** tells EF Core to load related entities **in the same query** using JOINs. This is the **recommended default** strategy.

### Basic Include

```csharp
// Loads Orders AND their Items in ONE query (using JOIN)
var orders = db.Orders
    .Include(o => o.Items)          // LEFT JOIN OrderItems ...
    .ToList();

// SQL generated:
// SELECT [o].*, [i].*
// FROM [Orders] AS [o]
// LEFT JOIN [OrderItems] AS [i] ON [o].[Id] = [i].[OrderId]
```

### ThenInclude for Nested Navigation

```csharp
// Order -> Items -> Product (two levels deep)
var orders = db.Orders
    .Include(o => o.Items)
        .ThenInclude(i => i.Product)    // joins Product table too
    .ToList();
```

### Multiple Includes

```csharp
// Load multiple navigation properties
var orders = db.Orders
    .Include(o => o.Items)          // collection navigation
    .Include(o => o.Customer)       // reference navigation
    .Include(o => o.ShippingAddress) // another reference
    .ToList();
```

### Filtered Include (.NET 5+)

```csharp
// Only include items that cost more than $50
var orders = db.Orders
    .Include(o => o.Items.Where(i => i.Price > 50))
    .ToList();
```

```ad-note
title: Filtered Include limitations
Filtered Include supports `Where`, `OrderBy`, `OrderByDescending`, `ThenBy`, `ThenByDescending`, `Skip`, and `Take`. It applies the filter to the related collection **in the SQL query**. The filtered result is what gets loaded — not the full collection.
```

### When Eager Loading Gets Heavy

Eager loading with many `Include` calls can generate large JOIN queries that return redundant data (the "Cartesian explosion" problem). If an order has 10 items and 5 notes, a single query with both includes returns 50 rows.

EF Core mitigates this with **split queries**:
```csharp
// Instead of one massive JOIN, EF sends separate queries
var orders = db.Orders
    .Include(o => o.Items)
    .Include(o => o.Notes)
    .AsSplitQuery()             // separate SELECT per Include
    .ToList();

// Generates:
// Query 1: SELECT * FROM Orders
// Query 2: SELECT * FROM OrderItems WHERE OrderId IN (...)
// Query 3: SELECT * FROM OrderNotes WHERE OrderId IN (...)
```

```ad-tip
title: When to use AsSplitQuery
Use `AsSplitQuery()` when you have multiple collection-type `Include` calls on the same query. For single includes or reference navigations, the default single query is fine.
```

---

## Strategy 2: Lazy Loading

**Lazy loading** automatically loads related data the first time you access a navigation property. It's convenient but dangerous.

### Setup

1. Install `Microsoft.EntityFrameworkCore.Proxies` NuGet package.
2. Enable in `DbContext` configuration:
```csharp
services.AddDbContext<AppDbContext>(options =>
    options.UseLazyLoadingProxies()     // enables proxy generation
           .UseSqlServer(connectionString));
```
3. Mark navigation properties as `virtual`:
```csharp
public class Order
{
    public int Id { get; set; }
    public virtual ICollection<OrderItem> Items { get; set; }  // must be virtual
    public virtual Customer Customer { get; set; }              // must be virtual
}
```

### How It Works

```csharp
var order = db.Orders.First();    // 1 query: loads the order only
var items = order.Items;           // 2nd query: triggered transparently by proxy
var name = order.Customer.Name;    // 3rd query: another transparent load
```

EF Core creates a runtime **proxy** subclass that overrides the `virtual` properties. When you access `order.Items`, the proxy intercepts the call and runs a `SELECT` if the data hasn't been loaded yet.

```ad-warning
title: Lazy loading causes N+1 by design
Lazy loading is the N+1 problem **built in as a feature**. Every navigation property access inside a loop becomes a separate database query. It's only safe when you access a small number of navigation properties outside of loops.
```

```ad-warning
title: Lazy loading after DbContext disposal
If you access a lazy-loaded navigation property after the `DbContext` has been disposed (e.g., after the HTTP request ends in ASP.NET Core), you get an `ObjectDisposedException`. This is a common source of bugs.
```

---

## Strategy 3: Explicit Loading

**Explicit loading** gives you manual control over when related data is loaded. You load it on demand using `Entry()`.

```csharp
var order = db.Orders.First();

// Explicitly load a collection navigation
db.Entry(order)
  .Collection(o => o.Items)
  .Load();

// Explicitly load a reference navigation
db.Entry(order)
  .Reference(o => o.Customer)
  .Load();
```

### Query Before Loading

You can apply filters before loading:
```csharp
// Only load expensive items
db.Entry(order)
  .Collection(o => o.Items)
  .Query()                              // returns IQueryable — you can filter
  .Where(i => i.Price > 100)
  .Load();
```

### Check If Loaded

```csharp
bool isLoaded = db.Entry(order)
    .Collection(o => o.Items)
    .IsLoaded;
```

---

## Comparison Table

| Feature | Eager Loading | Lazy Loading | Explicit Loading |
|---|---|---|---|
| **How** | `.Include()` / `.ThenInclude()` | Access `virtual` property | `Entry().Collection().Load()` |
| **When loaded** | With the parent query | On first access | When you call `Load()` |
| **SQL queries** | 1 (or split) | 1 per navigation access | 1 per `Load()` call |
| **N+1 risk** | No | **Yes** (the whole point) | Only if you call `Load()` in a loop |
| **Setup required** | None | Proxies package + `virtual` | None |
| **Works after dispose** | Yes (data already loaded) | No (`ObjectDisposedException`) | No (needs `DbContext`) |
| **Best for** | Most scenarios | Prototyping / small datasets | Loading conditionally based on runtime logic |

---

## Recommendations

1. **Default to eager loading** (`Include`) — it's explicit, predictable, and avoids N+1.
2. **Use `AsSplitQuery()`** when you have multiple collection includes to avoid Cartesian explosion.
3. **Avoid lazy loading in production** — it hides database access behind property accessors, making performance problems invisible until they hit production.
4. **Use explicit loading** when you need to conditionally load related data based on runtime logic (e.g., load items only if the user expands an order detail).
5. **Use Select projections** as an alternative — projecting into DTOs with `.Select()` lets EF Core generate efficient SQL that only fetches the columns you need, avoiding the need for Include entirely. See [[LINQ to Entities]].

---

## See Also

- [[LINQ to Entities]] — query fundamentals and Select projections
- [[Change Tracking]] — how loaded entities are tracked
- [[CRUD Operations]] — full create/read/update/delete patterns
