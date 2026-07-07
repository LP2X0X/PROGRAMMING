---
tags:
  - csharp
  - ef-core
  - concurrency
  - database
---

## The Problem: Lost Updates

Imagine two users both load the same car record from the database:

```text
Time 1:  User A reads Car { Id=1, Price=25000 }
Time 2:  User B reads Car { Id=1, Price=25000 }
Time 3:  User A changes Price to 27000, saves  --> UPDATE Cars SET Price=27000 WHERE Id=1
Time 4:  User B changes Price to 23000, saves  --> UPDATE Cars SET Price=23000 WHERE Id=1
```

User A's change is **silently overwritten**. This is the **lost update** problem, and it is one of the most common data integrity bugs in multi-user applications.

---

## Optimistic vs Pessimistic Concurrency

| Approach | How It Works | Locks? | EF Core Support |
|---|---|---|---|
| **Optimistic** | Don't lock. Let everyone read and write freely, but **detect conflicts at save time**. If a conflict is found, throw an exception. | No | Yes (built-in) |
| **Pessimistic** | Lock the row when a user starts editing. No one else can modify it until the lock is released. | Yes | No (must use raw SQL) |

EF Core uses **optimistic concurrency** exclusively. The idea is that conflicts are rare, so it's cheaper to detect them than to lock rows preemptively.

> [!ad-note] When Pessimistic Makes Sense
> If your application has very high contention on specific rows (e.g., inventory counts during a flash sale), optimistic concurrency may cause too many retries. In that case, you'd use raw SQL with `SELECT ... WITH (UPDLOCK, ROWLOCK)` or move the logic to a stored procedure. EF Core does not provide a built-in API for pessimistic locking.

---

## How Optimistic Concurrency Works in EF Core

The mechanism is simple: EF Core includes the **original value** of the concurrency token in the `WHERE` clause of the `UPDATE` statement. If someone else changed the row, the `WHERE` won't match, zero rows are affected, and EF throws `DbUpdateConcurrencyException`.

```text
Normal UPDATE:
  UPDATE Cars SET Price = 27000 WHERE Id = 1
  --> Always updates (even if someone else changed the row)

Concurrency-aware UPDATE:
  UPDATE Cars SET Price = 27000 WHERE Id = 1 AND Price = 25000
  --> Only updates if Price is still what we originally read
  --> 0 rows affected = conflict detected!
```

---

## Option 1: [ConcurrencyCheck] Attribute

Mark individual properties as concurrency tokens. EF will include their **original values** in the WHERE clause:

```csharp
public class Car
{
    public int Id { get; set; }
    public string Name { get; set; }

    [ConcurrencyCheck]   // include original Price in WHERE clause of UPDATE
    public decimal Price { get; set; }
}
```

Generated SQL when saving a price change:

```sql
UPDATE Cars
SET Price = @newPrice
WHERE Id = @id AND Price = @originalPrice;
-- If 0 rows affected --> DbUpdateConcurrencyException
```

**Fluent API equivalent:**

```csharp
modelBuilder.Entity<Car>()
    .Property(c => c.Price)
    .IsConcurrencyToken();
```

> [!ad-note] When to Use [ConcurrencyCheck]
> Use this when you want to protect **specific columns** from lost updates. If someone changes the car's `Name` but not its `Price`, there is no conflict on `Price` and the update succeeds. This gives you fine-grained control.

---

## Option 2: [Timestamp] / Row Version (Recommended)

A **row version** is a `byte[]` column that the database automatically increments on every UPDATE. It catches **any** change to the row, not just specific columns.

### Entity Setup

```csharp
public class Car
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }

    [Timestamp]                    // SQL Server: rowversion column
    public byte[] RowVersion { get; set; }
}
```

**Fluent API equivalent:**

```csharp
modelBuilder.Entity<Car>()
    .Property(c => c.RowVersion)
    .IsRowVersion();             // configures as concurrency token + value generated on add/update
```

### What Happens in the Database

SQL Server creates a `rowversion` (formerly `timestamp`) column. Every time any column in the row changes, SQL Server automatically updates this value.

```sql
-- EF generates:
UPDATE Cars
SET Name = @newName, Price = @newPrice
WHERE Id = @id AND RowVersion = @originalRowVersion;

-- If 0 rows affected, the row was modified since we read it
```

> [!ad-note] [Timestamp] vs [ConcurrencyCheck]
> - `[Timestamp]` detects **any change** to the row, regardless of which column changed. One token covers all columns.
> - `[ConcurrencyCheck]` only detects changes to the **specific marked property**.
> 
> For most applications, `[Timestamp]` is the better default because it prevents all lost updates, not just on selected columns.

---

## Handling DbUpdateConcurrencyException

When EF detects a conflict, it throws `DbUpdateConcurrencyException`. You must catch it and decide what to do:

### Strategy 1: "Last Write Wins" (Force Overwrite)

Reload the current database values, then re-apply your changes:

```csharp
bool saved = false;
while (!saved)
{
    try
    {
        context.SaveChanges();
        saved = true;
    }
    catch (DbUpdateConcurrencyException ex)
    {
        foreach (var entry in ex.Entries)
        {
            // Get current values from the database
            var dbValues = entry.GetDatabaseValues();

            if (dbValues == null)
            {
                // The row was deleted by another user
                throw new Exception("The record was deleted by another user.");
            }

            // Overwrite the original values with DB values
            // (so the next UPDATE's WHERE clause uses the current RowVersion)
            entry.OriginalValues.SetValues(dbValues);
        }
        // Loop retries SaveChanges with the updated RowVersion
    }
}
```

### Strategy 2: "Client Wins" (Keep User's Values)

```csharp
catch (DbUpdateConcurrencyException ex)
{
    foreach (var entry in ex.Entries)
    {
        var dbValues = entry.GetDatabaseValues();
        // Keep the user's current values, but update OriginalValues
        // so the WHERE clause matches
        entry.OriginalValues.SetValues(dbValues);
    }
    context.SaveChanges();  // saves user's values with updated RowVersion
}
```

### Strategy 3: "Show the Conflict" (Let the User Decide)

This is the most user-friendly approach for important data:

```csharp
catch (DbUpdateConcurrencyException ex)
{
    var entry = ex.Entries.Single();
    var currentValues = entry.CurrentValues;         // what the user wants to save
    var originalValues = entry.OriginalValues;       // what was read from DB initially
    var databaseValues = entry.GetDatabaseValues();  // what's in DB right now

    // Present all three versions to the user:
    // "You read Price=25000, changed it to 27000,
    //  but someone else already changed it to 23000.
    //  Which value do you want?"

    // After user decides, update the entry and retry:
    entry.OriginalValues.SetValues(databaseValues);
    context.SaveChanges();
}
```

---

## Complete Working Example

```csharp
public class Car
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }

    [Timestamp]
    public byte[] RowVersion { get; set; }
}

// --- Usage ---
public void UpdateCarPrice(int carId, decimal newPrice)
{
    using var context = new AppDbContext();

    var car = context.Cars.Find(carId);
    car.Price = newPrice;

    try
    {
        context.SaveChanges();
        Console.WriteLine("Saved successfully.");
    }
    catch (DbUpdateConcurrencyException ex)
    {
        var entry = ex.Entries.Single();
        var dbValues = entry.GetDatabaseValues();

        if (dbValues == null)
        {
            Console.WriteLine("Car was deleted by another user.");
            return;
        }

        var dbCar = (Car)dbValues.ToObject();
        Console.WriteLine($"Conflict! DB has Price={dbCar.Price}, you tried Price={newPrice}");

        // Auto-resolve: accept DB values, discard user's change
        entry.OriginalValues.SetValues(dbValues);
        entry.CurrentValues.SetValues(dbValues);  // discard user's changes
        context.SaveChanges();
    }
}
```

---

## Key Methods on EntityEntry

| Method / Property | Returns | Description |
|---|---|---|
| `entry.CurrentValues` | PropertyValues | The values the application is trying to save |
| `entry.OriginalValues` | PropertyValues | The values originally read from the database |
| `entry.GetDatabaseValues()` | PropertyValues | Queries the DB for the **current** row values |
| `entry.Reload()` | void | Reloads the entity from DB, discarding local changes |

> [!ad-warning] `entry.Reload()` Overwrites Everything
> Calling `Reload()` replaces both `CurrentValues` and `OriginalValues` with the database's current state. Any unsaved changes the user made are lost. Only use this if you intentionally want to discard local changes.

---

## Concurrency with Disconnected Entities (Web APIs)

In web applications, the entity is read in one request and saved in another. The `RowVersion` must survive the round-trip:

```csharp
// GET endpoint: return the RowVersion to the client
public CarDto GetCar(int id)
{
    var car = context.Cars.Find(id);
    return new CarDto
    {
        Id = car.Id,
        Name = car.Name,
        Price = car.Price,
        RowVersion = car.RowVersion   // send to client as Base64 or byte[]
    };
}

// PUT endpoint: client sends back the original RowVersion
public IActionResult UpdateCar(CarDto dto)
{
    var car = new Car
    {
        Id = dto.Id,
        Name = dto.Name,
        Price = dto.Price,
        RowVersion = dto.RowVersion   // original value from the GET
    };

    context.Cars.Update(car);

    // EF uses dto.RowVersion in the WHERE clause
    // If the row changed since the GET, DbUpdateConcurrencyException
    context.SaveChanges();
    return Ok();
}
```

> [!ad-warning] Do Not Forget the RowVersion in DTOs
> If you don't include `RowVersion` in your API response and request, the client has no way to detect conflicts. The concurrency check silently stops working and you're back to "last write wins."

---

## Related Topics

- [[Fluent API Configuration]] -- configure concurrency tokens with `.IsConcurrencyToken()` and `.IsRowVersion()`
- [[Entity Classes]] -- where `[Timestamp]` and `[ConcurrencyCheck]` attributes are placed
- [[Migrations Overview]] -- adding a `RowVersion` column requires a new migration
