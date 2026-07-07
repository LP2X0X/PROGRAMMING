---
tags: [csharp, ef-core, fluent-api, relationships]
---

## Overview

- The Fluent API provides the most explicit and complete way to configure [[Relationships|relationships]] between entities.
- While conventions and the `[ForeignKey]` / `[InverseProperty]` Data Annotations handle simple cases, the Fluent API is essential for:
  - Specifying delete behavior
  - Configuring many-to-many join tables
  - Resolving ambiguous relationships
  - Configuring composite foreign keys
  - Controlling which side is principal vs dependent

### The Fluent API Relationship Pattern

- Every relationship configuration follows the same pattern:

```
modelBuilder.Entity<Dependent>()
    .HasOne/HasMany(...)       // This entity's side of the relationship
    .WithOne/WithMany(...)     // The other entity's side
    .HasForeignKey(...)        // Which property is the FK
    .OnDelete(...)             // Delete behavior
```

- `HasOne` + `WithMany` = **One-to-Many** (configured from the "many" side)
- `HasOne` + `WithOne` = **One-to-One**
- `HasMany` + `WithMany` = **Many-to-Many**

---

## One-to-Many (Most Common)

- One parent entity has many children. Each child belongs to exactly one parent.
- Example: a `Make` (Honda, Toyota) has many `Car`s. Each `Car` belongs to one `Make`.

### Entity Classes

```csharp
public class Make
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Collection navigation — "a make has many cars"
    public ICollection<Car> Cars { get; set; } = new List<Car>();
}

public class Car
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Foreign key property
    public int MakeId { get; set; }
    
    // Reference navigation — "this car belongs to one make"
    public Make Make { get; set; }
}
```

### Fluent API Configuration

```csharp
modelBuilder.Entity<Car>()
    .HasOne(c => c.Make)                  // Car → one Make
    .WithMany(m => m.Cars)               // Make → many Cars
    .HasForeignKey(c => c.MakeId)        // FK is Car.MakeId
    .OnDelete(DeleteBehavior.Cascade);   // delete Make → delete its Cars
```

- **Reading it as a sentence**: "A Car has one Make, and that Make has many Cars, linked by Car.MakeId, with cascade delete."

### Configuring from Either Side

- You can configure the same relationship from either the principal or dependent side. These are equivalent:

```csharp
// From the dependent (Car) — more common, more intuitive
modelBuilder.Entity<Car>()
    .HasOne(c => c.Make)
    .WithMany(m => m.Cars)
    .HasForeignKey(c => c.MakeId);

// From the principal (Make) — same result
modelBuilder.Entity<Make>()
    .HasMany(m => m.Cars)
    .WithOne(c => c.Make)
    .HasForeignKey(c => c.MakeId);
```

```ad-tip
title: Pick One Side and Be Consistent
Both produce the same migration. Most teams configure from the dependent side (the entity that has the FK) because the FK column lives there, making it feel more natural. Whatever you choose, be consistent across your project.
```

### Without Navigation Property on One Side

- You don't always need navigation properties on both sides:

```csharp
// No collection navigation on Make — you just don't need it
modelBuilder.Entity<Car>()
    .HasOne(c => c.Make)
    .WithMany()                     // empty — no Make.Cars collection
    .HasForeignKey(c => c.MakeId);

// No reference navigation on Car — less common but valid
modelBuilder.Entity<Make>()
    .HasMany(m => m.Cars)
    .WithOne()                      // empty — no Car.Make reference
    .HasForeignKey(c => c.MakeId);
```

---

## One-to-One

- Each entity on both sides references exactly one of the other.
- Example: each `Car` has exactly one `CarDetail` (extended specifications), and each `CarDetail` belongs to exactly one `Car`.

### Entity Classes

```csharp
public class Car
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Reference navigation
    public CarDetail Detail { get; set; }
}

public class CarDetail
{
    public int Id { get; set; }
    public string EngineType { get; set; }
    public int Horsepower { get; set; }
    
    // Foreign key — points to Car
    public int CarId { get; set; }
    
    // Reference navigation
    public Car Car { get; set; }
}
```

### Fluent API Configuration

```csharp
modelBuilder.Entity<Car>()
    .HasOne(c => c.Detail)                         // Car → one CarDetail
    .WithOne(d => d.Car)                           // CarDetail → one Car
    .HasForeignKey<CarDetail>(d => d.CarId);       // FK is on CarDetail
```

- Notice `HasForeignKey<CarDetail>` — the generic type parameter specifies **which side holds the FK**. This is required for one-to-one relationships because either side could theoretically hold the FK.

```ad-warning
title: You Must Specify the Dependent Side
In one-to-one relationships, EF Core cannot always determine which entity should hold the foreign key. If you don't specify `HasForeignKey<TDependent>(...)`, EF Core might pick the wrong side, or fail entirely. Always be explicit.
```

### Shared Primary Key Pattern

- A common alternative for one-to-one relationships: the dependent uses the principal's PK as both its PK and FK.

```csharp
public class CarDetail
{
    public int CarId { get; set; }   // both PK and FK — same value as Car.Id
    public string EngineType { get; set; }
    public int Horsepower { get; set; }
    
    public Car Car { get; set; }
}
```

```csharp
modelBuilder.Entity<Car>()
    .HasOne(c => c.Detail)
    .WithOne(d => d.Car)
    .HasForeignKey<CarDetail>(d => d.CarId);

// CarDetail.CarId is both PK and FK
modelBuilder.Entity<CarDetail>()
    .HasKey(d => d.CarId);
```

- This is more storage-efficient (no separate `Id` column on the dependent) and makes the one-to-one nature very clear in the schema.

---

## Many-to-Many

- Both sides can reference many of the other.
- Example: a `Car` can have many `Feature`s, and each `Feature` can apply to many `Car`s.
- Requires a **join table** in the database to hold the FK pairs.

### EF Core 5+ — Skip Navigation (No Explicit Join Entity)

```csharp
public class Car
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Skip navigation — no explicit join entity needed
    public ICollection<Feature> Features { get; set; } = new List<Feature>();
}

public class Feature
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    public ICollection<Car> Cars { get; set; } = new List<Car>();
}
```

```csharp
modelBuilder.Entity<Car>()
    .HasMany(c => c.Features)
    .WithMany(f => f.Cars)
    .UsingEntity(j => j.ToTable("CarFeatures"));   // customize join table name
```

- EF Core automatically creates the join table with FK columns to both sides.
- **Without `UsingEntity`**, EF Core names the join table `CarFeature` (alphabetical concatenation of entity names).

**Resulting SQL:**
```sql
CREATE TABLE CarFeatures (
    CarsId     int NOT NULL,
    FeaturesId int NOT NULL,
    PRIMARY KEY (CarsId, FeaturesId),
    FOREIGN KEY (CarsId) REFERENCES Cars(Id) ON DELETE CASCADE,
    FOREIGN KEY (FeaturesId) REFERENCES Features(Id) ON DELETE CASCADE
);
```

### Explicit Join Entity (When Join Table Has Extra Data)

- When the relationship itself carries data (e.g., date added, priority, configuration value), you need an explicit join entity class:

```csharp
public class CarFeature
{
    public int CarId { get; set; }
    public Car Car { get; set; }
    
    public int FeatureId { get; set; }
    public Feature Feature { get; set; }
    
    // Extra data on the relationship
    public bool IsStandard { get; set; }      // is this a standard or optional feature?
    public decimal? AdditionalCost { get; set; }
}
```

```csharp
// Composite PK on the join entity
modelBuilder.Entity<CarFeature>()
    .HasKey(cf => new { cf.CarId, cf.FeatureId });

// Car → CarFeature (one-to-many)
modelBuilder.Entity<CarFeature>()
    .HasOne(cf => cf.Car)
    .WithMany(c => c.CarFeatures)
    .HasForeignKey(cf => cf.CarId);

// Feature → CarFeature (one-to-many)
modelBuilder.Entity<CarFeature>()
    .HasOne(cf => cf.Feature)
    .WithMany(f => f.CarFeatures)
    .HasForeignKey(cf => cf.FeatureId);
```

- The explicit join entity is really just **two one-to-many relationships** meeting at the join table, with a composite PK.
- Navigation properties on `Car` and `Feature` change from `ICollection<Feature>` to `ICollection<CarFeature>`:

```csharp
public class Car
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<CarFeature> CarFeatures { get; set; } = new List<CarFeature>();
}

public class Feature
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<CarFeature> CarFeatures { get; set; } = new List<CarFeature>();
}
```

```ad-note
title: Skip Navigation vs Explicit Join — When to Choose
Use **skip navigation** (automatic join) when the relationship has no extra data — just two FKs. Use an **explicit join entity** the moment you need any additional columns on the join table (dates, flags, sort order, amounts). This is extremely common in real-world applications. The automatic join table cannot be extended after the fact without changing your model significantly.
```

---

## Delete Behaviors

- **Delete behavior** controls what happens to dependent (child) entities when the principal (parent) entity is deleted.
- Configured with `.OnDelete(DeleteBehavior.X)`.

### Delete Behavior Reference

| Behavior | When Parent Is Deleted | FK Nullable? | Effect |
|---|---|---|---|
| `Cascade` | Yes | No (required) | Dependent entities are automatically deleted |
| `SetNull` | Yes | Yes (optional) | FK column is set to NULL |
| `Restrict` | Yes | Either | Exception thrown — deletion is blocked |
| `NoAction` | Yes | Either | Database engine decides (usually throws FK violation) |
| `ClientSetNull` | Yes | Yes | FK set to null in EF change tracker, then saved |

### Defaults

| Relationship Type | FK Nullable? | Default Behavior |
|---|---|---|
| Required | No (`int MakeId`) | `Cascade` |
| Optional | Yes (`int? MakeId`) | `ClientSetNull` |

### Configuring Delete Behavior

```csharp
// Cascade — deleting a Make also deletes all its Cars
modelBuilder.Entity<Car>()
    .HasOne(c => c.Make)
    .WithMany(m => m.Cars)
    .HasForeignKey(c => c.MakeId)
    .OnDelete(DeleteBehavior.Cascade);

// Restrict — cannot delete a Make that still has Cars
modelBuilder.Entity<Car>()
    .HasOne(c => c.Make)
    .WithMany(m => m.Cars)
    .HasForeignKey(c => c.MakeId)
    .OnDelete(DeleteBehavior.Restrict);

// SetNull — deleting a Make sets Car.MakeId to NULL (FK must be nullable)
modelBuilder.Entity<Car>()
    .HasOne(c => c.Make)
    .WithMany(m => m.Cars)
    .HasForeignKey(c => c.NullableMakeId)     // int? NullableMakeId
    .OnDelete(DeleteBehavior.SetNull);
```

```ad-warning
title: Cascade Delete Can Be Dangerous
`Cascade` is the default for required relationships. This means deleting a `Make` silently deletes **every** `Car` belonging to it — potentially hundreds or thousands of records. In production systems with important data, consider `Restrict` instead. It forces you to explicitly handle dependent records before deleting, preventing accidental mass deletion.
```

```ad-tip
title: Cascade Cycles
SQL Server rejects cascade delete when it detects a cycle — where deleting entity A cascades to B, which cascades back to A (directly or indirectly). You'll see: "Introducing FOREIGN KEY constraint may cause cycles or multiple cascade paths." Fix by changing one relationship in the cycle to `Restrict` or `NoAction`.
```

### ClientSetNull vs SetNull

| Behavior | Where It Happens | Behavior When Entity Not Tracked |
|---|---|---|
| `SetNull` | In the database (SQL SET NULL) | Still works — database handles it |
| `ClientSetNull` | In EF Core's change tracker (C# side) | **Fails** — EF Core can't set null on entities it doesn't know about |

- `ClientSetNull` only works for entities that are loaded (tracked) by the DbContext. If a parent is deleted and its children are not loaded, the database gets a DELETE without a SET NULL, causing an FK violation.
- `SetNull` creates an `ON DELETE SET NULL` constraint in the database, so it works regardless of whether EF Core has the entities loaded.

---

## Required vs Optional Relationships

- **Required relationship**: the FK is NOT NULL — the dependent cannot exist without a principal.
- **Optional relationship**: the FK is nullable — the dependent can exist without a principal.

```csharp
// Required — every Car MUST have a Make
modelBuilder.Entity<Car>()
    .HasOne(c => c.Make)
    .WithMany(m => m.Cars)
    .HasForeignKey(c => c.MakeId)       // int MakeId — non-nullable
    .IsRequired();                       // explicit, though convention already infers this

// Optional — a Car may or may not have an assigned Make
modelBuilder.Entity<Car>()
    .HasOne(c => c.Make)
    .WithMany(m => m.Cars)
    .HasForeignKey(c => c.NullableMakeId)   // int? NullableMakeId — nullable
    .IsRequired(false);
```

- EF Core infers required/optional from the FK property type:
  - `int MakeId` (non-nullable) = required relationship
  - `int? MakeId` (nullable) = optional relationship
- `.IsRequired()` / `.IsRequired(false)` lets you override the inference explicitly.

---

## Composite Foreign Keys

```csharp
// When the principal has a composite PK, the dependent needs a composite FK
modelBuilder.Entity<CarFeatureOption>()
    .HasOne(o => o.CarFeature)
    .WithMany(cf => cf.Options)
    .HasForeignKey(o => new { o.CarId, o.FeatureId });    // composite FK
```

- The composite FK must match the composite PK of the principal entity in both type and order.

---

## Configuring Without Navigation Properties

- Sometimes entities are related in the database but you don't want navigation properties in C# (to keep entities lean or to break circular references):

```csharp
// No navigation properties at all — configure by type
modelBuilder.Entity<Car>()
    .HasOne<Make>()                      // no lambda — just the type
    .WithMany()
    .HasForeignKey(c => c.MakeId);
```

- This is also useful when the FK property exists but you deliberately omit the navigation for architectural reasons (e.g., bounded context boundaries in DDD).

---

## Multiple Relationships Between the Same Entities

- When two entities have more than one relationship, EF Core can't automatically determine which navigation matches which FK. Use the Fluent API to be explicit:

```csharp
public class Game
{
    public int Id { get; set; }
    
    public int HomeTeamId { get; set; }
    public Team HomeTeam { get; set; }
    
    public int AwayTeamId { get; set; }
    public Team AwayTeam { get; set; }
}

public class Team
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    public ICollection<Game> HomeGames { get; set; }
    public ICollection<Game> AwayGames { get; set; }
}
```

```csharp
modelBuilder.Entity<Game>()
    .HasOne(g => g.HomeTeam)
    .WithMany(t => t.HomeGames)
    .HasForeignKey(g => g.HomeTeamId)
    .OnDelete(DeleteBehavior.Restrict);     // can't cascade both — would cycle

modelBuilder.Entity<Game>()
    .HasOne(g => g.AwayTeam)
    .WithMany(t => t.AwayGames)
    .HasForeignKey(g => g.AwayTeamId)
    .OnDelete(DeleteBehavior.Restrict);
```

```ad-note
title: SQL Server Cascade Cycle Warning
When two (or more) FKs from the same dependent point to the same principal table, SQL Server rejects cascading deletes on more than one of them (it detects a "multiple cascade paths" situation). Set at least one to `Restrict` or `NoAction`.
```

---

## Relationship Configuration Cheat Sheet

| Method | Purpose |
|---|---|
| `.HasOne(x => x.Nav)` | This entity has one related entity (reference navigation) |
| `.HasMany(x => x.Nav)` | This entity has many related entities (collection navigation) |
| `.WithOne(x => x.Nav)` | The other side has one (completing one-to-one or many-to-one) |
| `.WithMany(x => x.Nav)` | The other side has many (completing one-to-many) |
| `.HasForeignKey(x => x.FK)` | Specify the FK property |
| `.HasForeignKey<TDependent>(x => x.FK)` | Specify FK with explicit dependent type (for one-to-one) |
| `.HasPrincipalKey(x => x.AK)` | FK targets an alternate key instead of the PK |
| `.IsRequired()` / `.IsRequired(false)` | Required or optional relationship |
| `.OnDelete(DeleteBehavior.X)` | Delete behavior when principal is deleted |
| `.UsingEntity(...)` | Configure the join table (many-to-many) |

---

## See Also

- [[Fluent API Overview]] — the big picture and IEntityTypeConfiguration pattern
- [[Relationships]] — fundamentals of EF Core relationships, key terms, and conventions
- [[Key and Index Configuration]] — primary keys, composite keys, and alternate keys
- [[Entity Classes]] — entity configuration conventions and Data Annotations
- [[Inheritance Mapping]] — how relationships interact with TPH/TPT/TPC hierarchies
