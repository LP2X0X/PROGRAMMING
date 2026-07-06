---
tags: [csharp, ef-core, change-tracking, performance]
---

- EF Core's change tracker watches **every entity loaded from the database** through a `DbContext`. When you call `SaveChanges()`, EF Core compares the current state of tracked entities against their original values and generates the appropriate `INSERT`, `UPDATE`, or `DELETE` SQL. Understanding change tracking is essential for both correctness and performance.

---

## The Five Entity States

Every tracked entity is in exactly one of these states:

| State | Meaning | What `SaveChanges()` does |
|---|---|---|
| **Added** | New entity, doesn't exist in DB yet | `INSERT` |
| **Unchanged** | Loaded from DB, no modifications | Nothing |
| **Modified** | Loaded from DB, one or more properties changed | `UPDATE` (only changed columns) |
| **Deleted** | Marked for deletion | `DELETE` |
| **Detached** | Not tracked by this `DbContext` at all | Nothing |

---

## How States Change

```csharp
using var db = new AppDbContext();

// Query result -> Unchanged
var car = db.Cars.First(c => c.Id == 1);
// State: Unchanged

// Modify a property -> Modified
car.Color = "Blue";
// State: Modified (EF detects the change)

// Add a new entity -> Added
var newCar = new Car { Make = "Toyota", Model = "Camry" };
db.Cars.Add(newCar);
// State: Added

// Mark for deletion -> Deleted
db.Cars.Remove(car);
// State: Deleted

// SaveChanges sends INSERT, UPDATE, DELETE as needed
db.SaveChanges();
// After SaveChanges: Added -> Unchanged, Modified -> Unchanged, Deleted -> Detached
```

### State Transition Diagram

```
   +-----------+    query     +------------+
   | Detached  |  -------->  | Unchanged  |
   +-----------+             +------------+
        ^                     |          |
        |                     | modify   | Remove()
        |              +------v---+  +---v-----+
        |              | Modified |  | Deleted  |
        |              +----------+  +---------+
        |                     |          |
        |     SaveChanges()   |          |
        +---------------------+----------+
                              
   +-------+    Add()    +------------+
   |  new   | -------->  |   Added    |
   +-------+             +------------+
                               |
                    SaveChanges()
                               |
                          +----v-------+
                          | Unchanged  |
                          +------------+
```

---

## Inspecting Entity State

### Single Entity

```csharp
var entry = db.Entry(car);

// Check current state
EntityState state = entry.State;
Console.WriteLine(state); // e.g., EntityState.Modified

// Check which properties changed
foreach (var prop in entry.Properties)
{
    if (prop.IsModified)
    {
        Console.WriteLine($"{prop.Metadata.Name}: " +
            $"'{prop.OriginalValue}' -> '{prop.CurrentValue}'");
    }
}
```

### All Tracked Entities

```csharp
// See everything the change tracker is watching
foreach (var entry in db.ChangeTracker.Entries())
{
    Console.WriteLine($"{entry.Entity.GetType().Name}: {entry.State}");
}

// Filter by state
var modifiedEntities = db.ChangeTracker.Entries()
    .Where(e => e.State == EntityState.Modified)
    .ToList();
```

---

## Manually Setting State

Sometimes you need to tell EF Core what state an entity should be in — typically when working with **disconnected entities** (entities created outside the current `DbContext` scope, like data received from an API request).

```csharp
// Force an entity to a specific state
db.Entry(car).State = EntityState.Modified;
// This tells EF: "treat this as an existing entity that has been changed"
// SaveChanges will generate UPDATE for ALL columns

db.Entry(car).State = EntityState.Added;
// This tells EF: "treat this as a new entity"
// SaveChanges will generate INSERT
```

```ad-warning
title: Setting state to Modified updates ALL columns
When you manually set `State = EntityState.Modified`, EF Core marks **every** property as modified and generates an `UPDATE` that sets all columns. This is different from the normal flow where EF detects exactly which properties changed and only updates those.
```

---

## DetectChanges — When and How

EF Core uses **snapshot change detection**: when it loads an entity, it stores a copy of the original values. When `DetectChanges()` runs, it compares current values against the snapshot.

`DetectChanges()` is called **automatically** by:
- `SaveChanges()` / `SaveChangesAsync()`
- `Entry(entity)` (for that specific entity)
- `ChangeTracker.Entries()` 
- LINQ queries (to maintain consistency)

```ad-note
title: You rarely need to call DetectChanges manually
EF Core calls it for you at the right times. The only case you might call it manually is if you modify entities and then inspect the `ChangeTracker` without calling one of the auto-triggering methods. In practice, this is rare.
```

---

## AsNoTracking — Read-Only Performance

When you only need to **read** data and won't modify it, skip change tracking entirely:

```csharp
// Tracked (default) — EF stores snapshots, watches for changes
var cars = db.Cars.ToList();

// Not tracked — faster, less memory, no change detection
var cars = db.Cars.AsNoTracking().ToList();
```

### Performance Impact

| Scenario | Tracked | AsNoTracking |
|---|---|---|
| Memory per entity | Original values snapshot + entity | Entity only |
| DetectChanges cost | O(n) comparison on SaveChanges | Zero |
| Identity resolution | Yes (same PK = same object) | No (duplicate objects possible) |
| Can SaveChanges? | Yes | No (entity is Detached) |

```ad-tip
title: Use AsNoTracking for all read-only queries
In web APIs and read-heavy applications, most queries are read-only (display data, generate reports, etc.). Add `AsNoTracking()` to these queries for a meaningful performance improvement, especially when loading many entities.
```

### AsNoTrackingWithIdentityResolution

A middle ground: no change tracking, but if two JOINed rows reference the same entity (same PK), EF returns the **same object instance** instead of duplicates:

```csharp
var orders = db.Orders
    .Include(o => o.Customer)
    .AsNoTrackingWithIdentityResolution()
    .ToList();
// If two orders share the same customer, order1.Customer and order2.Customer
// point to the same Customer object in memory
```

---

## DbContext-Level NoTracking Default

If most of your queries are read-only, you can set `NoTracking` as the default:

```csharp
// In DbContext configuration or OnConfiguring
services.AddDbContext<AppDbContext>(options =>
{
    options.UseSqlServer(connectionString);
    options.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
});
```

Then opt **in** to tracking when you need it:
```csharp
var car = db.Cars.AsTracking().First(c => c.Id == 1);
car.Color = "Red";
db.SaveChanges(); // works because we explicitly opted into tracking
```

---

## See Also

- [[DbContext]] — the unit-of-work that owns the change tracker
- [[CRUD Operations]] — how Add, Remove, and SaveChanges use change tracking
- [[LINQ to Entities]] — how queries populate tracked entities
- [[Loading Related Data]] — how Include affects tracking
