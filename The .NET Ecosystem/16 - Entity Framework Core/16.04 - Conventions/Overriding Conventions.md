---
tags: [csharp, ef-core, conventions, configuration]
---

## When to Override Conventions

Conventions are designed for the common case. You need to override them when:

- Your **database naming** doesn't match your C# naming (e.g., `snake_case` columns, prefixed table names, legacy schemas)
- You need **constrained string lengths** (convention defaults to `nvarchar(max)`)
- You need **composite primary keys** (conventions can't detect them)
- Your **foreign key property** doesn't follow the `{NavName}Id` pattern
- You need **query filters**, **indexes**, or **inheritance mapping** that have no convention equivalent
- You're working with a **legacy database** whose schema you can't change

---

## The Precedence Rule

EF Core applies configuration in this strict order -- later stages override earlier ones:

```
Convention  →  Data Annotations  →  Fluent API
(applied first)                    (applied last, always wins)
```

If **both** a Data Annotation and Fluent API configure the same thing, **Fluent API wins**. If only a Data Annotation is present, it overrides the convention. If neither is present, the convention applies.

```csharp
// All three approaches configure the same property:

// 1. Convention: Name → "Name" column, nvarchar(max), NOT NULL
public string Name { get; set; }

// 2. Data Annotation: Name → "Name" column, nvarchar(100), NOT NULL
[MaxLength(100)]
public string Name { get; set; }

// 3. Fluent API: Name → "ProductName" column, nvarchar(200), NOT NULL
entity.Property(p => p.Name)
    .HasColumnName("ProductName")
    .HasMaxLength(200);

// Result: "ProductName" column, nvarchar(200) -- Fluent API wins on everything it touches.
// MaxLength(100) from the annotation is overridden by HasMaxLength(200).
```

---

## Side-by-Side Override Examples

### Table Name

| Approach | Code | Result |
|---|---|---|
| **Convention** | `DbSet<Car> Cars { get; set; }` | Table: "Cars" |
| **Data Annotation** | `[Table("Vehicles")]` on the `Car` class | Table: "Vehicles" |
| **Fluent API** | `entity.ToTable("Vehicles")` | Table: "Vehicles" |

```csharp
// Convention -- table name comes from DbSet property name
public DbSet<Car> Cars { get; set; }                    // → "Cars"

// Data Annotation -- override with [Table]
[Table("Vehicles")]
public class Car
{
    public int Id { get; set; }
    public string Name { get; set; }
}

// Fluent API -- override in OnModelCreating
modelBuilder.Entity<Car>(entity =>
{
    entity.ToTable("Vehicles");                          // → "Vehicles"
});
```

### Column Name

| Approach | Code | Result |
|---|---|---|
| **Convention** | `public string FirstName { get; set; }` | Column: "FirstName" |
| **Data Annotation** | `[Column("first_name")]` | Column: "first_name" |
| **Fluent API** | `.HasColumnName("first_name")` | Column: "first_name" |

```csharp
// Convention
public string FirstName { get; set; }                   // → "FirstName" column

// Data Annotation
[Column("first_name")]
public string FirstName { get; set; }                   // → "first_name" column

// Fluent API
entity.Property(c => c.FirstName)
    .HasColumnName("first_name");                        // → "first_name" column
```

### Required (NOT NULL)

| Approach | Code | Result |
|---|---|---|
| **Convention** | `public string Name { get; set; }` (with NRT enabled) | NOT NULL |
| **Convention** | `public string? Name { get; set; }` | NULL allowed |
| **Data Annotation** | `[Required]` on a `string?` property | NOT NULL (overrides nullability) |
| **Fluent API** | `.IsRequired()` | NOT NULL |

```csharp
// Convention -- NRT controls nullability
public string Name { get; set; }                         // NOT NULL (NRT enabled)
public string? Name { get; set; }                        // Nullable

// Data Annotation -- force NOT NULL even on nullable type
[Required]
public string? MiddleName { get; set; }                  // NOT NULL (annotation overrides)

// Fluent API
entity.Property(c => c.MiddleName)
    .IsRequired();                                        // NOT NULL
```

### Max Length

| Approach | Code | Result |
|---|---|---|
| **Convention** | `public string Name { get; set; }` | `nvarchar(max)` |
| **Data Annotation** | `[MaxLength(100)]` | `nvarchar(100)` |
| **Fluent API** | `.HasMaxLength(100)` | `nvarchar(100)` |

```csharp
// Convention -- no max length, defaults to nvarchar(max)
public string Name { get; set; }                         // → nvarchar(max)

// Data Annotation
[MaxLength(100)]
public string Name { get; set; }                         // → nvarchar(100)

// Fluent API
entity.Property(c => c.Name)
    .HasMaxLength(100);                                   // → nvarchar(100)
```

```ad-warning
title: nvarchar(max) Is Almost Never What You Want
The convention default of `nvarchar(max)` can hurt performance -- SQL Server treats `max` columns differently (they can't be used as index keys, and queries against them are slower). **Always** set explicit max lengths on string properties in production code. This is the most common convention you'll override.
```

### Primary Key

| Approach | Code | Result |
|---|---|---|
| **Convention** | Property named `Id` or `{ClassName}Id` | PK |
| **Data Annotation** | `[Key]` on any property | PK |
| **Fluent API** | `.HasKey(x => x.PropertyName)` | PK |

```csharp
// Convention -- "Id" or "{ClassName}Id" is automatically the PK
public int Id { get; set; }                              // PK by convention

// Data Annotation -- mark any property as PK
[Key]
public int ProductCode { get; set; }                     // PK (would NOT be detected by convention)

// Fluent API
entity.HasKey(p => p.ProductCode);                       // PK
```

### Column Data Type

| Approach | Code | Result |
|---|---|---|
| **Convention** | `public decimal Price { get; set; }` | `decimal(18,2)` |
| **Data Annotation** | `[Column(TypeName = "decimal(10,4)")]` | `decimal(10,4)` |
| **Fluent API** | `.HasPrecision(10, 4)` or `.HasColumnType("decimal(10,4)")` | `decimal(10,4)` |

```csharp
// Convention
public decimal Price { get; set; }                       // → decimal(18,2)

// Data Annotation
[Column(TypeName = "decimal(10,4)")]
public decimal Price { get; set; }                       // → decimal(10,4)

// Fluent API
entity.Property(p => p.Price)
    .HasPrecision(10, 4);                                 // → decimal(10,4)
```

### Excluding a Property (Not Mapped)

| Approach | Code | Result |
|---|---|---|
| **Convention** | Read-only properties (`=>`) excluded automatically | Not in DB |
| **Data Annotation** | `[NotMapped]` | Not in DB |
| **Fluent API** | `.Ignore(x => x.Property)` | Not in DB |

```csharp
// Convention -- computed property with no setter is automatically excluded
public string FullName => $"{First} {Last}";             // ❌ Not a column

// Data Annotation -- explicitly exclude a read-write property
[NotMapped]
public int TemporaryCalculation { get; set; }            // ❌ Not a column

// Fluent API
entity.Ignore(c => c.TemporaryCalculation);              // ❌ Not a column
```

---

## Configuration Capabilities Comparison

This table shows **which approach can configure what**. This is the key reference for deciding which approach to use.

| Configuration | Convention | Data Annotations | Fluent API |
|---|---|---|---|
| **Table name** | Auto (from DbSet) | `[Table("name")]` | `.ToTable("name")` |
| **Table schema** | Auto (default) | `[Table("name", Schema = "x")]` | `.ToTable("name", "x")` |
| **Column name** | Auto (from property) | `[Column("name")]` | `.HasColumnName("name")` |
| **Column data type** | Auto (from .NET type) | `[Column(TypeName = "x")]` | `.HasColumnType("x")` |
| **Primary key** | Auto (`Id` / `{Class}Id`) | `[Key]` | `.HasKey(...)` |
| **Composite key** | -- | -- | `.HasKey(x => new { x.A, x.B })` |
| **Required (NOT NULL)** | Auto (NRT) | `[Required]` | `.IsRequired()` |
| **Max length** | -- | `[MaxLength(n)]` | `.HasMaxLength(n)` |
| **Precision/Scale** | -- | `[Precision(p, s)]` (EF Core 6+) | `.HasPrecision(p, s)` |
| **Not mapped** | Auto (no setter) | `[NotMapped]` | `.Ignore(...)` |
| **Foreign key** | Auto (naming) | `[ForeignKey("prop")]` | `.HasForeignKey(...)` |
| **Inverse property** | Auto (pairing) | `[InverseProperty("prop")]` | `.WithMany(...)` / `.WithOne(...)` |
| **Cascade behavior** | Auto (required vs optional) | -- | `.OnDelete(...)` |
| **Default value** | -- | -- | `.HasDefaultValue(v)` |
| **Default SQL** | -- | -- | `.HasDefaultValueSql("x")` |
| **Computed column** | -- | -- | `.HasComputedColumnSql("x")` |
| **Index** | Auto (on FK) | `[Index]` (EF Core 5+) | `.HasIndex(...)` |
| **Unique index** | -- | `[Index(IsUnique = true)]` | `.HasIndex(...).IsUnique()` |
| **Query filter** | -- | -- | `.HasQueryFilter(x => ...)` |
| **Value conversion** | -- | -- | `.HasConversion<T>()` |
| **Inheritance mapping** | Auto (TPH) | -- | `.UseTphMappingStrategy()`, etc. |
| **Owned type** | -- | `[Owned]` | `.OwnsOne(...)` / `.OwnsMany(...)` |
| **Concurrency token** | -- | `[ConcurrencyCheck]` | `.IsConcurrencyToken()` |
| **Row version** | -- | `[Timestamp]` | `.IsRowVersion()` |
| **Backing field** | Auto (naming) | -- | `.HasField("_fieldName")` |
| **Seed data** | -- | -- | `.HasData(...)` |
| **Table splitting** | -- | -- | Multiple entities → `.ToTable("same")` |

```ad-note
title: Key Observations from This Table
1. **Conventions** cover the basics: table/column discovery, naming, PKs, FKs, nullability, and indexes on FKs.
2. **Data Annotations** add simple overrides: names, types, required, max length, basic indexes.
3. **Fluent API** is the only option for: composite keys, query filters, default values, computed columns, value conversions, seed data, table splitting, and advanced relationship configuration.
4. If it's not in the first two columns, you **must** use Fluent API.
```

---

## Practical Decision Guide

```ad-tip
title: The Rule of Thumb
**Start with conventions.** Add **Data Annotations** for simple, property-level overrides (names, lengths, required). Use **Fluent API** for everything else.
```

### When to Use Each

**Stick with conventions when:**
- Your C# naming matches your desired database naming
- PKs follow the `Id` / `{ClassName}Id` pattern
- FKs follow the `{NavName}Id` pattern
- You don't need constrained string lengths (prototyping or non-production code)

**Use Data Annotations when:**
- You need `[Required]`, `[MaxLength]`, `[Table]`, `[Column]`, `[Key]`, or `[NotMapped]`
- The same attributes serve double duty with ASP.NET validation (e.g., `[Required]` validates both in EF and in MVC model binding)
- You want configuration visible right on the entity class

**Use Fluent API when:**
- You need composite keys, query filters, default values, computed columns, or value conversions
- You want to keep entity classes completely clean (no EF-specific attributes in your domain model)
- You're configuring complex relationships (custom FK names, cascade behavior, many-to-many with payload)
- You want all database configuration centralized in one place

### Mixing Approaches

You can freely mix all three approaches. They're not mutually exclusive:

```csharp
// Entity class uses both conventions and Data Annotations
[Table("Products")]                                     // Data Annotation
public class Product
{
    public int Id { get; set; }                         // Convention (PK)

    [Required]                                          // Data Annotation
    [MaxLength(150)]                                    // Data Annotation
    public string Name { get; set; }

    public decimal Price { get; set; }                  // Convention (decimal(18,2))
    public int CategoryId { get; set; }                 // Convention (FK)
    public Category Category { get; set; }              // Convention (navigation)
}

// Fluent API adds what annotations can't do
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Product>(entity =>
    {
        entity.Property(p => p.Price)
            .HasPrecision(10, 4);                        // Override convention decimal precision

        entity.HasQueryFilter(p => !p.IsDeleted);        // Only available in Fluent API
    });
}
```

---

## Bulk Convention Overrides with IEntityTypeConfiguration

When you have many entities to configure, putting everything in `OnModelCreating` gets messy. Use **`IEntityTypeConfiguration<T>`** to organize Fluent API configuration into separate classes:

```csharp
// Separate configuration class for Product
public class ProductConfiguration : IEntityTypeConfiguration<Product>
{
    public void Configure(EntityTypeBuilder<Product> builder)
    {
        builder.ToTable("Products");
        builder.Property(p => p.Name).HasMaxLength(150).IsRequired();
        builder.Property(p => p.Price).HasPrecision(10, 4);
        builder.HasQueryFilter(p => !p.IsDeleted);
    }
}

// Register all configurations in OnModelCreating
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Apply a single configuration
    modelBuilder.ApplyConfiguration(new ProductConfiguration());

    // OR: Apply ALL IEntityTypeConfiguration classes from an assembly
    modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
}
```

```ad-tip
title: ApplyConfigurationsFromAssembly Is a Huge Win
`ApplyConfigurationsFromAssembly` scans the assembly for all classes that implement `IEntityTypeConfiguration<T>` and applies them automatically. This means you just create a new configuration class and it's picked up -- no manual registration needed. This is the recommended approach for any project with more than a handful of entities.
```

---

## Custom Model Conventions (EF Core 7+)

Starting with EF Core 7, you can define **custom conventions** that apply rules to your entire model. This is different from Fluent API (which configures specific entities/properties) -- custom conventions apply a rule **across all entities**.

```csharp
// Custom convention: all string properties get max length 256 instead of nvarchar(max)
protected override void ConfigureConventions(ModelConfigurationBuilder configurationBuilder)
{
    configurationBuilder.Properties<string>()
        .HaveMaxLength(256);                            // All strings → nvarchar(256)

    configurationBuilder.Properties<decimal>()
        .HavePrecision(18, 4);                          // All decimals → decimal(18,4)
}
```

This is especially useful for:
- Setting **default max lengths** on all strings (avoid `nvarchar(max)` everywhere)
- Setting **default precision** on all decimals
- Applying **value converters** to all properties of a specific type (e.g., all `DateOnly` properties)

```ad-note
title: ConfigureConventions vs OnModelCreating
`ConfigureConventions` runs BEFORE `OnModelCreating`. It sets **model-wide defaults** that individual Fluent API calls in `OnModelCreating` can still override per-entity. Think of it as "convention replacement" -- you're changing what the default is, not configuring specific entities.
```

---

## See Also

- [[Conventions Overview]] -- what conventions are and the full convention summary table
- [[Table and Column Conventions]] -- the default naming and typing rules you're overriding
- [[Key and Relationship Conventions]] -- the default key and relationship rules you're overriding
- [[Entity Classes]] -- how entity classes work with all three configuration approaches
- [[Fluent API Configuration]] -- in-depth Fluent API reference
- [[Data Annotations Overview]] -- in-depth Data Annotations reference
