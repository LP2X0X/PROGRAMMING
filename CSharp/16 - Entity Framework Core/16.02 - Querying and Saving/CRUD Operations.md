---
tags: [csharp, ef-core, crud, saving]
---

- EF Core follows the **Unit of Work** pattern: you make changes to tracked entities in memory, then call `SaveChanges()` to persist them all to the database in a single transaction. This note covers the full Create, Read, Update, Delete lifecycle.

---

## Create (INSERT)

### Single Entity

```csharp
var car = new Car
{
    Make = "Honda",
    Model = "Civic",
    Year = 2024,
    Color = "Silver"
};

db.Cars.Add(car);           // State -> Added
db.SaveChanges();            // INSERT INTO Cars (...) VALUES (...)

// After SaveChanges, car.Id is populated with the DB-generated value
Console.WriteLine(car.Id);   // e.g., 42
```

### Multiple Entities

```csharp
var cars = new List<Car>
{
    new Car { Make = "Toyota", Model = "Camry", Year = 2024 },
    new Car { Make = "Ford", Model = "Mustang", Year = 2023 }
};

db.Cars.AddRange(cars);     // both -> Added
db.SaveChanges();            // two INSERT statements in one transaction
```

```ad-note
title: Add via DbContext vs DbSet
`db.Cars.Add(car)` and `db.Add(car)` do the same thing. The `DbContext.Add()` version infers the `DbSet` from the entity type. Use whichever reads better.
```

---

## Read (SELECT)

Reading is covered in detail in [[LINQ to Entities]]. The essentials:

```csharp
// All rows
var allCars = db.Cars.ToList();

// Filtered
var redCars = db.Cars.Where(c => c.Color == "Red").ToList();

// Single by PK (checks cache first!)
var car = db.Cars.Find(42);

// First match
var firstHonda = db.Cars.FirstOrDefault(c => c.Make == "Honda");
```

---

## Update (UPDATE)

### The Standard Pattern — Query, Modify, Save

```csharp
// 1. Load the entity (tracked, state = Unchanged)
var car = db.Cars.First(c => c.Id == 42);

// 2. Modify properties (state automatically becomes Modified)
car.Color = "Blue";
car.Year = 2025;

// 3. Save (EF generates UPDATE for only the changed columns)
db.SaveChanges();
// UPDATE Cars SET Color = 'Blue', Year = 2025 WHERE Id = 42
```

EF Core detects **exactly which properties changed** and generates an `UPDATE` that only includes those columns.

### Disconnected Update (API Scenario)

When the entity comes from outside the `DbContext` (e.g., from an HTTP request body), you need to tell EF about it:

```csharp
// carDto came from an API request — not tracked
var car = new Car
{
    Id = 42,          // must have the PK
    Make = "Honda",
    Model = "Civic",
    Color = "Blue",
    Year = 2025
};

db.Cars.Update(car);     // State -> Modified (ALL properties marked as changed)
db.SaveChanges();         // UPDATE Cars SET Make=..., Model=..., Color=..., Year=... WHERE Id=42
```

```ad-warning
title: Update() marks ALL properties as modified
`Update()` is a blunt tool — it generates an UPDATE that sets every column, even if the value didn't actually change. For large entities, this is wasteful. Prefer the query-modify-save pattern when possible, or manually set only the changed properties:

```csharp
var car = new Car { Id = 42 };
db.Cars.Attach(car);              // State -> Unchanged (treated as existing)
car.Color = "Blue";                // State -> Modified (only Color is marked changed)
db.SaveChanges();                  // UPDATE Cars SET Color = 'Blue' WHERE Id = 42
```
```

---

## Delete (DELETE)

### Standard Pattern — Query Then Remove

```csharp
var car = db.Cars.First(c => c.Id == 42);  // load it (tracked)
db.Cars.Remove(car);                        // State -> Deleted
db.SaveChanges();                            // DELETE FROM Cars WHERE Id = 42
```

### Delete Without Loading (Stub Entity)

```csharp
// Create a "stub" with just the PK — no database query needed
var car = new Car { Id = 42 };
db.Cars.Remove(car);              // Attach + mark Deleted
db.SaveChanges();                  // DELETE FROM Cars WHERE Id = 42
```

```ad-note
title: Cascade deletes
If the entity has related data (e.g., an Order with OrderItems), EF Core's cascade delete behavior depends on your model configuration. By default, required relationships cascade: deleting the principal (Order) also deletes dependents (OrderItems). Optional relationships set the FK to null.
```

---

## Attach vs Add

| Method | Sets state to | Use when |
|---|---|---|
| `Add(entity)` | **Added** | Brand new entity, needs INSERT |
| `Attach(entity)` | **Unchanged** | Existing entity, you want to track it without marking it modified |
| `Update(entity)` | **Modified** | Existing entity, you want to update ALL columns |
| `Remove(entity)` | **Deleted** | Existing entity, you want to delete it |

```csharp
// Attach: "I know this exists in the DB, just start tracking it"
var car = new Car { Id = 42, Make = "Honda" };
db.Cars.Attach(car);                // State: Unchanged — no SQL yet
car.Color = "Red";                   // State: Modified (only Color)
db.SaveChanges();                    // UPDATE Cars SET Color='Red' WHERE Id=42
```

---

## SaveChanges — The Core Method

`SaveChanges()` does the following:
1. Calls `DetectChanges()` to find all modifications (see [[Change Tracking]]).
2. Wraps everything in a **database transaction**.
3. Generates and executes SQL for all Added, Modified, and Deleted entities.
4. **Commits** the transaction if all SQL succeeds, or **rolls back** if any fails.
5. Updates the state of all entities (Added -> Unchanged, Deleted -> Detached, etc.).
6. Returns the **number of state entries written** to the database.

```csharp
try
{
    int rowsAffected = db.SaveChanges();
    Console.WriteLine($"{rowsAffected} row(s) saved.");
}
catch (DbUpdateConcurrencyException ex)
{
    // Another user modified the same row — handle conflict
}
catch (DbUpdateException ex)
{
    // Database constraint violation, connection error, etc.
    Console.WriteLine(ex.InnerException?.Message);
}
```

```ad-tip
title: SaveChanges is all-or-nothing
If you Add 3 entities and one fails (e.g., unique constraint violation), **none** of them are saved — the entire transaction rolls back. This is usually what you want. If you need partial saves, call `SaveChanges()` between operations or use explicit transactions. See [[Transactions]].
```

---

## The Set-Based Limitation

EF Core's traditional approach sends **individual SQL statements per entity**:

```csharp
// Adding 1000 cars
db.Cars.AddRange(thousandCars);
db.SaveChanges();
// Generates 1000 individual INSERT statements!
// INSERT INTO Cars (...) VALUES (...); -- repeated 1000 times
```

This is slow for bulk operations. Solutions:

### ExecuteUpdate / ExecuteDelete (.NET 7+)

Set-based operations that run directly in the database **without loading entities**:

```csharp
// Update all old cars in one SQL statement — no entities loaded
db.Cars
    .Where(c => c.Year < 2000)
    .ExecuteUpdate(setters => setters
        .SetProperty(c => c.IsClassic, true));
// UPDATE Cars SET IsClassic = 1 WHERE Year < 2000

// Delete all discontinued cars — no entities loaded
db.Cars
    .Where(c => c.IsDiscontinued)
    .ExecuteDelete();
// DELETE FROM Cars WHERE IsDiscontinued = 1
```

```ad-warning
title: ExecuteUpdate/ExecuteDelete bypass the change tracker
These methods execute SQL directly and **do not** update tracked entities. If you have a Car with `Id = 5` loaded and tracked, and `ExecuteUpdate` changes it in the database, the tracked entity still has the old values. Either reload it or call these before loading entities.
```

### For True Bulk Inserts

For inserting thousands of rows, consider:
- **EF Core Bulk Extensions** (third-party NuGet) — adds `BulkInsert`, `BulkUpdate`, `BulkDelete`
- **SqlBulkCopy** (ADO.NET) — the fastest option for SQL Server, uses the bulk copy protocol
- EF's own batching does help (it groups INSERT statements), but it's still per-row SQL

---

## Async Versions

Always use async in web applications:

```csharp
var car = await db.Cars.FindAsync(42);

await db.Cars.AddAsync(newCar);              // only needed for value generators
await db.SaveChangesAsync();

await db.Cars.Where(c => c.Year < 2000)
    .ExecuteUpdateAsync(s => s.SetProperty(c => c.IsClassic, true));

await db.Cars.Where(c => c.IsDiscontinued)
    .ExecuteDeleteAsync();
```

```ad-note
title: AddAsync is rarely needed
`AddAsync` is only necessary when using a custom value generator that needs async I/O (e.g., generating a unique ID from an external service). For normal usage, `Add` is fine even in async code — it's a synchronous in-memory operation. The async version is `SaveChangesAsync()`, which is always worth using.
```

---

## See Also

- [[LINQ to Entities]] — detailed querying patterns
- [[Change Tracking]] — how EF Core detects what to INSERT/UPDATE/DELETE
- [[Transactions]] — when you need more control over transaction boundaries
- [[Raw SQL]] — bypassing EF for bulk operations
- [[Parameterized Queries]] — EF Core always parameterizes, protecting against SQL injection
