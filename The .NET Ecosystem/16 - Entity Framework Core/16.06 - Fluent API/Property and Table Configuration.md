---
tags: [csharp, ef-core, fluent-api, properties, tables]
---

## Table Mapping

- By convention, EF Core uses the `DbSet<T>` property name as the table name (e.g., `DbSet<Car> Cars` maps to a "Cars" table).
- The Fluent API lets you override the table name, specify a schema, or do advanced mappings like table splitting.

### Basic Table Mapping

```csharp
modelBuilder.Entity<Car>(entity =>
{
    // Just the table name — uses the database's default schema (dbo in SQL Server)
    entity.ToTable("Vehicles");
    
    // Table name + schema
    entity.ToTable("Vehicles", "inventory");
});
```

**Resulting SQL:**
```sql
-- With schema
CREATE TABLE inventory.Vehicles ( ... );

-- Without schema (defaults to dbo in SQL Server)
CREATE TABLE dbo.Vehicles ( ... );
```

### Table Comments

```csharp
entity.ToTable("Vehicles", t => t.HasComment("Stores all vehicle inventory"));
```

- Adds a comment to the table in the database. Useful for documentation — some database tools display these comments.

---

## Property Configuration

- All property configuration starts from `.Property(x => x.Prop)`, which returns a `PropertyBuilder`.
- Every method on `PropertyBuilder` returns itself, so you chain calls fluently.

### Required (NOT NULL)

```csharp
entity.Property(c => c.Name)
    .IsRequired();          // column is NOT NULL
```

- Equivalent to the `[Required]` Data Annotation.
- For reference types with nullable reference types enabled, non-nullable properties (`string Name`) are already required by convention. `.IsRequired()` is explicit confirmation.
- For nullable properties (`string? Name`), calling `.IsRequired()` overrides the convention and makes the column NOT NULL.

### Max Length

```csharp
entity.Property(c => c.Name)
    .HasMaxLength(100);     // nvarchar(100) in SQL Server
```

- Equivalent to `[MaxLength(100)]` or `[StringLength(100)]`.
- Without this, `string` properties default to `nvarchar(max)` — which has performance implications for indexing and storage.

```ad-tip
title: Always Set MaxLength on String Properties
`nvarchar(max)` columns cannot be used as index keys in SQL Server (max key size is 900 bytes). If you plan to add an index or unique constraint to a string column, you **must** set a max length. Get in the habit of always specifying one — it's rare that a name or email truly needs unlimited length.
```

### Column Name

```csharp
entity.Property(c => c.Name)
    .HasColumnName("VehicleName");   // maps to "VehicleName" column instead of "Name"
```

- Useful when you're mapping to an existing database where column names don't match C# property names.
- Equivalent to `[Column("VehicleName")]`.

### Column Data Type

```csharp
entity.Property(c => c.Price)
    .HasColumnType("decimal(10,2)");   // exact SQL type
```

- Overrides EF Core's type inference. The string you pass is provider-specific SQL.
- Common uses:
  - `decimal(10,2)` — 10 total digits, 2 decimal places
  - `varchar(50)` — non-Unicode string (smaller than `nvarchar`)
  - `money` — SQL Server money type
  - `text` — large text storage

```ad-warning
title: decimal Precision Matters
EF Core defaults `decimal` to `decimal(18,2)` in SQL Server. If you need more precision (e.g., financial calculations with more decimal places) or less scale (e.g., quantities that are always whole numbers), configure it explicitly. Getting precision wrong can cause silent data truncation.
```

### Default Values

```csharp
// C# constant default value
entity.Property(c => c.IsActive)
    .HasDefaultValue(true);

// SQL expression default value — evaluated by the database
entity.Property(c => c.CreatedAt)
    .HasDefaultValueSql("GETUTCDATE()");
```

- `HasDefaultValue(value)` — the default is a C# constant embedded in the migration. The database assigns this value when no value is provided on INSERT.
- `HasDefaultValueSql("SQL expression")` — the default is a raw SQL expression evaluated by the database engine at insert time. Use this for timestamps, sequences, or any value that must be computed server-side.

```ad-warning
title: HasDefaultValue vs C# Property Initializers
`HasDefaultValue(true)` and `public bool IsActive { get; set; } = true;` are NOT the same thing. The C# initializer sets the value in your application code. `HasDefaultValue` sets a DEFAULT constraint in the database. If you INSERT a row outside of EF Core (raw SQL, another application), only the database default applies.
```

### Computed Columns

```csharp
// Virtual computed column — recalculated on every read
entity.Property(c => c.DisplayName)
    .HasComputedColumnSql("[Make] + ' ' + [Name]");

// Stored (persisted) computed column — calculated and stored on write
entity.Property(c => c.DisplayName)
    .HasComputedColumnSql("[Make] + ' ' + [Name]", stored: true);
```

- **Virtual** (default): the database recalculates the value each time the row is read. No extra storage, but uses CPU on read.
- **Stored**: the database calculates the value when the row is written and stores it. Uses disk space, but reads are free. Can also be indexed.

```ad-note
title: Computed Column SQL Is Provider-Specific
The SQL expression inside `HasComputedColumnSql(...)` is raw SQL for your target provider. `[Make] + ' ' + [Name]` works on SQL Server. For PostgreSQL you'd write `"Make" || ' ' || "Name"`. For SQLite, it's also `||`. Keep this in mind if you need to support multiple providers.
```

### Value Generation

```csharp
// Value generated automatically on INSERT (e.g., auto-increment identity)
entity.Property(c => c.Id)
    .ValueGeneratedOnAdd();

// Value generated on INSERT and UPDATE (e.g., rowversion, computed columns)
entity.Property(c => c.RowVersion)
    .ValueGeneratedOnAddOrUpdate();

// No auto-generation — must provide the value yourself
entity.Property(c => c.Code)
    .ValueGeneratedNever();
```

- `ValueGeneratedOnAdd()` — EF Core expects the database to generate this value on INSERT. For `int` PKs, this is identity/auto-increment by default. EF Core reads the generated value back after insert.
- `ValueGeneratedNever()` — EF Core will not expect the database to generate this value. You must set it in your C# code before calling `SaveChanges()`. Useful for natural keys (e.g., ISBN, country codes).

---

## Full Property Configuration Example

```csharp
modelBuilder.Entity<Car>(entity =>
{
    // Table mapping
    entity.ToTable("Vehicles", "inventory");
    
    // Property configurations
    entity.Property(c => c.Name)
        .IsRequired()                              // NOT NULL
        .HasMaxLength(100)                         // nvarchar(100)
        .HasColumnName("VehicleName");             // column name override
    
    entity.Property(c => c.Price)
        .HasColumnType("decimal(10,2)");           // exact SQL type
    
    entity.Property(c => c.IsActive)
        .HasDefaultValue(true);                    // DEFAULT 1
    
    entity.Property(c => c.CreatedAt)
        .HasDefaultValueSql("GETUTCDATE()");       // DEFAULT GETUTCDATE()
    
    entity.Property(c => c.DisplayName)
        .HasComputedColumnSql("[Make] + ' ' + [Name]");  // computed column
    
    entity.Ignore(c => c.TempCalculation);         // not mapped to DB
});
```

**Resulting SQL (SQL Server):**
```sql
CREATE TABLE inventory.Vehicles (
    Id              int            IDENTITY(1,1) NOT NULL PRIMARY KEY,
    VehicleName     nvarchar(100)  NOT NULL,
    Price           decimal(10,2)  NOT NULL,
    IsActive        bit            NOT NULL DEFAULT 1,
    CreatedAt       datetime2      NOT NULL DEFAULT GETUTCDATE(),
    DisplayName     AS ([Make] + ' ' + [Name]),
    MakeId          int            NOT NULL,
    -- TempCalculation is NOT here — it was ignored
    FOREIGN KEY (MakeId) REFERENCES Makes(Id)
);
```

---

## Ignoring Properties

```csharp
entity.Ignore(c => c.TempCalculation);
```

- Fluent API equivalent of the `[NotMapped]` Data Annotation.
- The property exists in the C# class but EF Core completely ignores it — no column, no tracking, no queries.
- Common use cases:
  - Computed display properties (`FullName` derived from `FirstName` + `LastName`)
  - Properties used only in application logic (caching, temp state)
  - Properties from a base class that shouldn't be mapped

You can also ignore entire entity types at the model level:

```csharp
modelBuilder.Ignore<AuditLog>();  // EF Core won't create a table for AuditLog
```

---

## Shadow Properties

- **Shadow properties** are properties that exist in the EF Core model and the database but have **no corresponding property in the C# class**.
- EF Core manages them internally. You access their values through the `Entry` API.

### Defining Shadow Properties

```csharp
modelBuilder.Entity<Car>(entity =>
{
    // Define a shadow property — no "LastModified" property exists on the Car class
    entity.Property<DateTime>("LastModified");
    
    // Shadow properties can have all the same configuration
    entity.Property<string>("CreatedBy")
        .HasMaxLength(100);
});
```

### Accessing Shadow Property Values

```csharp
// Setting a shadow property value
var car = new Car { Name = "Civic", MakeId = 1 };
db.Cars.Add(car);
db.Entry(car).Property("LastModified").CurrentValue = DateTime.UtcNow;
db.Entry(car).Property("CreatedBy").CurrentValue = "admin";
await db.SaveChangesAsync();

// Querying by shadow property
var recentCars = db.Cars
    .Where(c => EF.Property<DateTime>(c, "LastModified") > DateTime.UtcNow.AddDays(-7))
    .ToList();
```

### When to Use Shadow Properties

- **Audit columns** (CreatedAt, ModifiedAt, CreatedBy) — keep audit infrastructure out of domain classes.
- **Foreign keys** where you don't want an explicit FK property on the entity. (EF Core creates shadow FK properties automatically when you have a navigation property but no FK property.)
- **Discriminator columns** in TPH inheritance — the discriminator is a shadow property by default.

```ad-tip
title: EF Core Already Creates Shadow Properties for You
When you define a navigation property without an explicit FK property, EF Core automatically creates a shadow FK property. For example, if `Car` has `public Make Make { get; set; }` but no `MakeId` property, EF Core creates a shadow property `MakeId` (type `int`) behind the scenes. You just can't access it via `car.MakeId` — you'd use `EF.Property<int>(car, "MakeId")` in queries or the Entry API.
```

---

## Global Query Filters

- A **query filter** is a LINQ predicate automatically appended to every query against an entity. It acts as an invisible global WHERE clause.
- The most common use case is **soft deletes** — entities are marked as deleted (`IsDeleted = true`) but never physically removed from the database.

### Defining a Query Filter

```csharp
modelBuilder.Entity<Car>(entity =>
{
    entity.HasQueryFilter(c => !c.IsDeleted);
});
```

- Now **every** query against `Car` automatically includes `WHERE IsDeleted = 0`:

```csharp
// Your code
var cars = db.Cars.ToList();

// Generated SQL
// SELECT * FROM Cars WHERE IsDeleted = 0
```

### Bypassing the Filter

```csharp
// When you need ALL records, including soft-deleted ones
var allCars = db.Cars
    .IgnoreQueryFilters()
    .ToList();

// Generated SQL
// SELECT * FROM Cars  (no WHERE IsDeleted = 0)
```

### Multi-Tenancy Filter

```csharp
// Filter by current tenant — every query is scoped to one tenant
modelBuilder.Entity<Car>(entity =>
{
    entity.HasQueryFilter(c => c.TenantId == _tenantId);
});
```

- `_tenantId` is a field on the DbContext, resolved from the current user's context (e.g., from an HTTP request header or JWT claim).
- EF Core captures the expression as a lambda, so the current value of `_tenantId` is used at query time, not at model-build time.

### Combining Multiple Filters

```csharp
// Only ONE HasQueryFilter call per entity is allowed — combine conditions with &&
entity.HasQueryFilter(c => !c.IsDeleted && c.TenantId == _tenantId);
```

```ad-warning
title: Only One Query Filter Per Entity
Calling `HasQueryFilter` multiple times on the same entity **replaces** the previous filter — it does NOT combine them. If you need both soft-delete and multi-tenancy filtering, combine them in a single expression with `&&`.
```

```ad-tip
title: Query Filters Apply to Includes Too
Query filters affect not just the top-level query but also navigation properties loaded with `.Include()`. If `Car` has a filter and you do `db.Makes.Include(m => m.Cars)`, only non-deleted cars are included. This is powerful but can be surprising — if you expect to see all related entities, remember to use `IgnoreQueryFilters()`.
```

---

## Precision and Scale for Decimal and DateTime

- EF Core 6+ introduced `.HasPrecision()` as a cleaner alternative to `.HasColumnType("decimal(x,y)")`:

```csharp
// Cleaner syntax (EF Core 6+)
entity.Property(c => c.Price)
    .HasPrecision(10, 2);    // 10 total digits, 2 after decimal point

// Equivalent older syntax
entity.Property(c => c.Price)
    .HasColumnType("decimal(10,2)");

// DateTime precision (SQL Server datetime2 precision: 0-7)
entity.Property(c => c.CreatedAt)
    .HasPrecision(3);        // datetime2(3) — millisecond precision
```

---

## Value Conversions

- **Value conversions** let you transform property values between C# and the database. EF Core stores one type in the database but your C# code works with a different type.

### Built-In Conversions

```csharp
// Store enum as string in the database instead of int
entity.Property(c => c.Status)
    .HasConversion<string>();    // CarStatus.Active → "Active" in DB

// Equivalent explicit converter
entity.Property(c => c.Status)
    .HasConversion(
        v => v.ToString(),                          // C# → DB
        v => (CarStatus)Enum.Parse(typeof(CarStatus), v)  // DB → C#
    );
```

### Common Value Conversion Scenarios

| C# Type | DB Type | Configuration |
|---|---|---|
| `enum` | `string` | `.HasConversion<string>()` — stores `"Active"` instead of `0` |
| `bool` | `int` | `.HasConversion<int>()` — some legacy DBs use 0/1 not bit |
| `DateOnly` | `DateTime` | Handled automatically in EF Core 8+ |
| `Uri` | `string` | `.HasConversion(v => v.ToString(), v => new Uri(v))` |
| `List<string>` | `string` (JSON) | Use JSON columns (EF Core 7+) or manual JSON serialization |

```ad-warning
title: Value Conversions and LINQ Queries
Value conversions happen in memory, not in SQL. If you convert an `enum` to a `string`, EF Core will translate `Where(c => c.Status == CarStatus.Active)` to `WHERE Status = 'Active'`. However, complex custom conversions may prevent EF Core from translating the expression to SQL, causing client-side evaluation or exceptions. Stick to simple, reversible conversions.
```

---

## Unicode and Collation

```csharp
// Force non-Unicode storage (varchar instead of nvarchar)
entity.Property(c => c.CountryCode)
    .IsUnicode(false);        // varchar(max) instead of nvarchar(max)

// Set collation for case-insensitive comparisons
entity.Property(c => c.Name)
    .UseCollation("SQL_Latin1_General_CP1_CI_AS");
```

- `IsUnicode(false)` reduces storage size when you know the data is ASCII-only (country codes, currency codes, etc.).
- Collation affects sorting and comparison behavior. The `CI` in `CI_AS` means **Case Insensitive** — useful when you want `WHERE Name = 'honda'` to match `'Honda'`.

---

## See Also

- [[Fluent API Overview]] — the big picture of how the Fluent API works and its entry points
- [[Entity Classes]] — conventions and Data Annotations for entity configuration
- [[Key and Index Configuration]] — primary keys, alternate keys, and indexes
- [[Relationship Configuration]] — configuring relationships between entities
- [[Concurrency Control]] — row versioning and concurrency tokens
- [[Seeding Data]] — `HasData()` for seed data in migrations
