---
tags: [csharp, ef-core, fluent-api, keys, indexes]
---

## Primary Keys

- By convention, a property named `Id` or `{ClassName}Id` is automatically treated as the primary key (see [[Entity Classes]] for convention details).
- The Fluent API lets you override this convention or configure composite keys.

### Single Primary Key

```csharp
modelBuilder.Entity<Car>(entity =>
{
    entity.HasKey(c => c.CarId);    // explicit single PK
});
```

- Useful when your PK property doesn't follow the `Id` / `{ClassName}Id` naming convention (e.g., `VehicleNumber`).

### Composite Primary Key

```csharp
modelBuilder.Entity<CarFeature>(entity =>
{
    // Composite PK — both columns together form the primary key
    entity.HasKey(cf => new { cf.CarId, cf.FeatureId });
});
```

- The anonymous object `new { cf.CarId, cf.FeatureId }` defines a PK spanning two columns.
- The resulting SQL creates a composite PRIMARY KEY constraint:

```sql
CREATE TABLE CarFeatures (
    CarId     int NOT NULL,
    FeatureId int NOT NULL,
    PRIMARY KEY (CarId, FeatureId)    -- composite PK
);
```

```ad-warning
title: Composite Keys Require Fluent API
There is **no** Data Annotation that can define a composite primary key. `[Key]` only works on a single property. You **must** use the Fluent API for composite keys. This is one of the most common reasons developers move from annotations to Fluent API.
```

### Named Primary Key Constraint

```csharp
entity.HasKey(c => c.Id)
    .HasName("PK_Vehicles");    // custom constraint name instead of auto-generated
```

- By default, EF Core generates constraint names like `PK_Cars`. Use `.HasName()` if your DBA requires a specific naming convention.

---

## Key Value Generation

- Controls how primary key values are generated — auto-increment, manual, or on every save.

```csharp
// Auto-increment (identity) — database generates the value on INSERT
entity.Property(c => c.Id)
    .ValueGeneratedOnAdd();

// Manual — you must set the Id before saving
entity.Property(c => c.Code)
    .ValueGeneratedNever();

// Generated on every INSERT and UPDATE (rare — used for rowversion/timestamp)
entity.Property(c => c.RowVersion)
    .ValueGeneratedOnAddOrUpdate();
```

### When to Use Each

| Method | Use Case | Example |
|---|---|---|
| `ValueGeneratedOnAdd()` | Surrogate keys — let the DB auto-assign | `int Id` with IDENTITY, `Guid Id` with NEWSEQUENTIALID |
| `ValueGeneratedNever()` | Natural keys — the value is meaningful, you provide it | ISBN, country codes, composite business keys |
| `ValueGeneratedOnAddOrUpdate()` | Concurrency tokens that update automatically | SQL Server `rowversion` / `timestamp` columns |

```ad-note
title: Convention Handles Most Cases
For `int`, `long`, and `Guid` primary keys, EF Core automatically configures `ValueGeneratedOnAdd()` by convention. You only need to call it explicitly when the convention doesn't apply — for example, when your PK is a `string` that you want the database to generate, or when you need to override the convention.
```

---

## Alternate Keys

- An **alternate key** is a unique constraint on one or more columns that can also serve as a **foreign key target**.
- This is distinct from a unique index — an alternate key is a true candidate key that EF Core can reference in relationships.

```csharp
modelBuilder.Entity<Car>(entity =>
{
    // Single alternate key
    entity.HasAlternateKey(c => c.VIN);
    
    // Composite alternate key
    entity.HasAlternateKey(c => new { c.LicensePlate, c.StateCode });
});
```

**Resulting SQL:**
```sql
-- Single alternate key
ALTER TABLE Cars ADD CONSTRAINT AK_Cars_VIN UNIQUE (VIN);

-- Composite alternate key
ALTER TABLE Cars ADD CONSTRAINT AK_Cars_LicensePlate_StateCode 
    UNIQUE (LicensePlate, StateCode);
```

### Alternate Key vs Unique Index

| Feature | Alternate Key | Unique Index |
|---|---|---|
| Uniqueness enforced | Yes | Yes |
| Can be FK target | **Yes** | No |
| Created with | `HasAlternateKey()` | `HasIndex().IsUnique()` |
| Database implementation | UNIQUE constraint | UNIQUE index |
| When to use | Another entity references this column as FK | Just need uniqueness |

```csharp
// Using an alternate key as a relationship target
modelBuilder.Entity<CarRegistration>()
    .HasOne(r => r.Car)
    .WithMany()
    .HasPrincipalKey(c => c.VIN)      // references Car.VIN instead of Car.Id
    .HasForeignKey(r => r.CarVIN);    // CarRegistration.CarVIN → Car.VIN
```

```ad-tip
title: Alternate Keys Are Rare in Practice
Most relationships reference the primary key. Alternate keys are useful when integrating with external systems that reference entities by a business identifier (VIN, SSN, SKU) rather than the internal surrogate key. If you just need uniqueness without FK targeting, use a unique index instead.
```

---

## Indexes

- Indexes speed up queries by allowing the database to find rows without scanning the entire table.
- EF Core automatically creates indexes on **foreign key columns**. You manually add indexes on columns frequently used in WHERE, ORDER BY, or JOIN clauses.

### Basic Index

```csharp
entity.HasIndex(c => c.Name);
```

```sql
CREATE INDEX IX_Cars_Name ON Cars (Name);
```

### Unique Index

```csharp
entity.HasIndex(c => c.VIN)
    .IsUnique();
```

```sql
CREATE UNIQUE INDEX IX_Cars_VIN ON Cars (VIN);
```

- A unique index also serves as a uniqueness constraint. Attempting to insert a duplicate value results in a database exception.

### Composite Index

```csharp
entity.HasIndex(c => new { c.MakeId, c.Year });
```

```sql
CREATE INDEX IX_Cars_MakeId_Year ON Cars (MakeId, Year);
```

- Column order matters in composite indexes. The index is most effective when queries filter by the **leftmost** columns first. `WHERE MakeId = 5 AND Year = 2024` uses the index fully. `WHERE Year = 2024` alone may not benefit from it (depends on the database engine).

```ad-warning
title: Column Order in Composite Indexes
A composite index on `(MakeId, Year)` is efficient for queries filtering on `MakeId` alone or `MakeId + Year`, but **not** for queries filtering only on `Year`. Think of it like a phone book sorted by last name, then first name — you can look up "Smith" easily, and "Smith, John" even faster, but finding all "Johns" across all last names requires a full scan. Order your composite index columns from most-filtered to least-filtered.
```

### Named Index

```csharp
entity.HasIndex(c => c.Name)
    .HasDatabaseName("IX_Cars_Name");
```

- Overrides the auto-generated name. Useful for matching an existing database schema or DBA naming conventions.

### Filtered Index (SQL Server)

```csharp
entity.HasIndex(c => c.VIN)
    .IsUnique()
    .HasFilter("[VIN] IS NOT NULL");
```

```sql
CREATE UNIQUE NONCLUSTERED INDEX IX_Cars_VIN 
    ON Cars (VIN) 
    WHERE [VIN] IS NOT NULL;
```

- A **filtered index** only includes rows matching the filter expression. This is powerful for:
  - **Unique-except-null**: Allow multiple NULLs but enforce uniqueness on non-null values.
  - **Sparse data**: If only 5% of rows have a non-null `DiscountCode`, a filtered index on `WHERE DiscountCode IS NOT NULL` is much smaller and faster.
  - **Active records**: `WHERE IsDeleted = 0` to index only active records.

```ad-note
title: Filtered Indexes Are Provider-Specific
Filtered indexes are a SQL Server feature. PostgreSQL has a similar concept called **partial indexes** (same Fluent API, different SQL). SQLite does not support filtered indexes. Check your provider's documentation.
```

### Included Columns (.NET 7+, SQL Server)

```csharp
entity.HasIndex(c => c.Name)
    .IncludeProperties(c => new { c.Price, c.Year });
```

```sql
CREATE INDEX IX_Cars_Name 
    ON Cars (Name) 
    INCLUDE (Price, Year);
```

- **Included columns** are stored in the index leaf nodes but are not part of the index key. This creates a **covering index** — the database can satisfy the query entirely from the index without looking up the base table.
- Use when a query frequently selects specific columns alongside the indexed column:

```csharp
// This query benefits from the covering index above
var results = db.Cars
    .Where(c => c.Name == "Civic")       // Name is the index key → seek
    .Select(c => new { c.Price, c.Year }) // Price, Year are INCLUDED → no table lookup
    .ToList();
```

```ad-tip
title: Covering Indexes Are a Huge Performance Win
Without `INCLUDE`, the database finds the matching rows via the index but must then do a **key lookup** (also called "bookmark lookup") to the base table to retrieve `Price` and `Year`. With `INCLUDE`, the data is already in the index — no extra I/O. For high-volume queries, this can cut query time significantly. Use SQL Server's execution plan to identify key lookups and fix them with `INCLUDE`.
```

### Descending Index (.NET 7+)

```csharp
entity.HasIndex(c => new { c.Year, c.Name })
    .IsDescending(true, false);   // Year DESC, Name ASC
```

- Useful when queries frequently use `ORDER BY Year DESC, Name ASC`.

---

## When to Add Indexes

### Good Candidates for Indexing

| Scenario | Why |
|---|---|
| Columns in WHERE clauses | Index seek instead of table scan |
| Columns in JOIN conditions | Faster join operations |
| Columns in ORDER BY | Avoid costly sort operations |
| Foreign key columns | **Already auto-indexed by EF Core** |
| Columns with high selectivity | Index is more useful when values are distinct (e.g., email, VIN) |

### Poor Candidates for Indexing

| Scenario | Why |
|---|---|
| Boolean columns (`IsActive`) | Only 2 distinct values — index provides little benefit |
| Columns rarely queried | Index is maintenance overhead with no read benefit |
| Very small tables | Full table scan is already fast |
| Frequently updated columns | Every UPDATE must also update the index |
| Wide columns (`nvarchar(max)`) | Cannot be indexed, or index would be very large |

```ad-warning
title: Don't Over-Index
Every index you add **slows down writes** (INSERT, UPDATE, DELETE) because the database must maintain the index alongside the table data. Each index also consumes disk space. Index strategically based on actual query patterns, not speculatively. Use query execution plans to identify missing indexes rather than guessing.
```

---

## Sequences

- A **sequence** is a database object that generates a sequence of numeric values. Unlike identity columns, sequences are not tied to a single table — multiple tables can share a sequence.

```csharp
// Define a sequence
modelBuilder.HasSequence<int>("OrderNumbers", "shared")
    .StartsAt(1000)
    .IncrementsBy(1);

// Use the sequence for a property
modelBuilder.Entity<Order>(entity =>
{
    entity.Property(o => o.OrderNumber)
        .HasDefaultValueSql("NEXT VALUE FOR shared.OrderNumbers");
});
```

- Sequences are useful when you need **guaranteed unique numbers across multiple tables** (e.g., a unified document numbering system) or when you need **non-identity PKs** (e.g., for TPC inheritance — see [[Inheritance Mapping]]).

---

## See Also

- [[Fluent API Overview]] — the Fluent API entry points and IEntityTypeConfiguration pattern
- [[Property and Table Configuration]] — property-level configuration (types, defaults, computed columns)
- [[Relationship Configuration]] — foreign keys and navigation property configuration
- [[Entity Classes]] — conventions for primary keys and foreign keys
- [[Inheritance Mapping]] — TPC strategy requires sequences for globally unique PKs
