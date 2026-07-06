---
tags: [csharp, ef-core, data-annotations, fundamentals]
---

## Overview

- **Data Annotations** are C# attributes you place on entity classes and their properties to tell EF Core how to map them to the database.
- They come from two namespaces in the .NET base class library — they are **not** part of the EF Core NuGet package itself, so they work even in projects that don't reference EF Core (e.g., shared DTOs validated by ASP.NET).
- Data Annotations sit in the **middle tier** of EF Core's configuration hierarchy: they override conventions but are themselves overridden by [[Fluent API Overview|Fluent API]] configuration.

### Namespaces

```csharp
using System.ComponentModel.DataAnnotations;        // [Key], [Required], [MaxLength], [StringLength], [ConcurrencyCheck], [Timestamp], etc.
using System.ComponentModel.DataAnnotations.Schema;  // [Table], [Column], [NotMapped], [ForeignKey], [InverseProperty], [DatabaseGenerated], etc.
```

```ad-tip
Both namespaces live in the `System.ComponentModel.DataAnnotations` assembly. In modern .NET (5+), this assembly is included automatically. In .NET Framework / .NET Standard projects you may need to add a NuGet reference to `System.ComponentModel.Annotations`.
```

---

## Dual Purpose — EF Core Mapping + ASP.NET Validation

- Data Annotations serve **two independent roles** depending on which framework reads them:

| Consumer | What It Does With Annotations |
| --- | --- |
| **EF Core** | Reads annotations at model-build time to configure the database schema (column types, nullability, keys, etc.) |
| **ASP.NET (MVC / Razor Pages / Web API)** | Reads annotations at model-binding time to validate incoming data (`ModelState.IsValid`) |

- Some annotations affect **both** systems, some affect **only one**:

| Annotation | EF Core Effect | ASP.NET Validation Effect |
| --- | --- | --- |
| `[Required]` | Column is `NOT NULL` | Field must have a value |
| `[MaxLength(n)]` | Column is `nvarchar(n)` | **None** |
| `[StringLength(n)]` | Column is `nvarchar(n)` | Max length validated |
| `[Range(min, max)]` | **None** (EF ignores it) | Range validated |
| `[EmailAddress]` | **None** | Email format validated |
| `[Phone]` | **None** | Phone format validated |
| `[RegularExpression]` | **None** | Regex validated |
| `[ConcurrencyCheck]` | Included in WHERE clause for optimistic concurrency | **None** |
| `[Timestamp]` | Maps to `rowversion` / concurrency token | **None** |
| `[Key]` | Marks primary key | **None** |
| `[Column]` | Sets column name / type | **None** |
| `[Table]` | Sets table name | **None** |
| `[NotMapped]` | Excludes from DB | **None** |

```ad-warning
title: Common Misconception
Developers sometimes assume `[Range]`, `[EmailAddress]`, or `[RegularExpression]` will create CHECK constraints in the database. **They don't.** EF Core ignores all validation-only annotations. If you need a database-level constraint, use [[Fluent API Overview|Fluent API]] with `HasCheckConstraint()` or raw SQL in a migration.
```

---

## Configuration Precedence

EF Core builds its internal model by layering three configuration sources. Later sources override earlier ones:

```
Convention  →  Data Annotations  →  Fluent API
(lowest)        (middle)             (highest)
```

1. **[[Conventions Overview|Conventions]]** — EF Core's built-in rules (e.g., a property named `Id` becomes the PK, `string` maps to `nvarchar(max)`).
2. **Data Annotations** — Attributes on entity classes and properties override conventions.
3. **[[Fluent API Overview|Fluent API]]** — Code in `OnModelCreating` overrides everything else.

```ad-note
If the same aspect is configured by both a Data Annotation and the Fluent API, **the Fluent API wins**. This means you can use annotations as defaults and selectively override with Fluent API where needed.
```

### Example of Precedence in Action

```csharp
// Convention: table name = DbSet property name ("Cars")
// Data Annotation: overrides to "Vehicles"
[Table("Vehicles")]
public class Car
{
    public int Id { get; set; }

    // Convention: column name = property name ("FullName"), type = nvarchar(max)
    // Data Annotation: overrides column name and length
    [Column("vehicle_name")]
    [MaxLength(200)]
    public string FullName { get; set; }
}

// Fluent API (in OnModelCreating): could override the table name again
// modelBuilder.Entity<Car>().ToTable("tbl_Cars");  // → this wins over [Table("Vehicles")]
```

---

## When to Use Data Annotations vs Fluent API

| Scenario | Recommended Approach | Why |
| --- | --- | --- |
| Simple NOT NULL, max-length, column/table name | **Data Annotations** | Config is visible right on the property — easy to read |
| You want validation + DB mapping from one attribute | **Data Annotations** | `[Required]` and `[StringLength]` do double duty |
| Entity classes are shared with non-EF projects (DTOs, Blazor models) | **Data Annotations** | No dependency on `Microsoft.EntityFrameworkCore` |
| Composite primary keys | **Fluent API** | `[Key]` alone cannot define composite keys in EF Core |
| Many-to-many with payload (join entity config) | **Fluent API** | Annotations can't configure join tables |
| Global query filters (`HasQueryFilter`) | **Fluent API** | No annotation equivalent |
| Table-per-hierarchy / TPT / TPC inheritance mapping | **Fluent API** | Annotations have limited inheritance support |
| Owned types, value conversions, seed data | **Fluent API** | No annotation equivalents |
| You want entity classes to stay "clean" (no attributes) | **Fluent API** | All config lives in `OnModelCreating` |

```ad-tip
**Practical rule of thumb**: Start with conventions. Add Data Annotations for simple, property-level config that benefits from being visible on the class. Use Fluent API for anything complex, composite, or EF-specific. Many real-world projects use a mix of both.
```

---

## Quick Reference — All Common Annotations

| Annotation | Namespace | Purpose | Details |
| --- | --- | --- | --- |
| `[Key]` | DataAnnotations | Mark property as primary key | See [[Key and Relationship Annotations]] |
| `[Required]` | DataAnnotations | NOT NULL + validation | See [[Validation and Concurrency Annotations]] |
| `[MaxLength(n)]` | DataAnnotations | Max column length (EF only) | See [[Table and Column Annotations]] |
| `[StringLength(n)]` | DataAnnotations | Max column length + validation | See [[Table and Column Annotations]] |
| `[Table("name")]` | Schema | Override table name | See [[Table and Column Annotations]] |
| `[Column("name")]` | Schema | Override column name / type | See [[Table and Column Annotations]] |
| `[NotMapped]` | Schema | Exclude from database | See [[Table and Column Annotations]] |
| `[ForeignKey("prop")]` | Schema | Specify FK property explicitly | See [[Key and Relationship Annotations]] |
| `[InverseProperty("prop")]` | Schema | Resolve ambiguous navigations | See [[Key and Relationship Annotations]] |
| `[DatabaseGenerated(...)]` | Schema | Control value generation | See [[Table and Column Annotations]] |
| `[ConcurrencyCheck]` | DataAnnotations | Optimistic concurrency on a column | See [[Validation and Concurrency Annotations]] |
| `[Timestamp]` | DataAnnotations | Row version (`byte[]`) concurrency token | See [[Validation and Concurrency Annotations]] |

---

## Complete Example — Decorated Entity Class

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

[Table("Products", Schema = "catalog")]              // table name + schema
public class Product
{
    [Key]                                              // explicit PK (not needed if property is "Id")
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int ProductId { get; set; }

    [Required(ErrorMessage = "Product name is required")]  // NOT NULL + validation
    [MaxLength(200)]                                        // nvarchar(200)
    public string Name { get; set; }

    [Column("product_description")]                    // custom column name
    [StringLength(2000)]                               // nvarchar(2000) + validation
    public string Description { get; set; }

    [Column(TypeName = "decimal(10,2)")]               // exact SQL type
    public decimal Price { get; set; }

    [NotMapped]                                        // not stored in DB
    public string DisplayText => $"{Name} - ${Price:F2}";

    [ConcurrencyCheck]                                 // optimistic concurrency
    public DateTime LastModified { get; set; }

    // FK relationship
    [ForeignKey("Category")]
    public int CategoryId { get; set; }
    public Category Category { get; set; }
}

public class Category
{
    public int Id { get; set; }

    [Required]
    [MaxLength(100)]
    public string Name { get; set; }

    public ICollection<Product> Products { get; set; } = new List<Product>();
}
```

### Resulting SQL (SQL Server)

```sql
CREATE TABLE catalog.Products (
    ProductId            int             IDENTITY(1,1) PRIMARY KEY,
    Name                 nvarchar(200)   NOT NULL,
    product_description  nvarchar(2000)  NULL,
    Price                decimal(10,2)   NOT NULL,
    LastModified         datetime2       NOT NULL,
    CategoryId           int             NOT NULL,
    FOREIGN KEY (CategoryId) REFERENCES Categories(Id)
);
-- Note: DisplayText is NOT in the table (NotMapped)
-- Note: LastModified is included in UPDATE WHERE clauses (ConcurrencyCheck)
```

---

## Cross-References

- [[Table and Column Annotations]] — deep dive into `[Table]`, `[Column]`, `[NotMapped]`, `[DatabaseGenerated]`, `[MaxLength]`
- [[Key and Relationship Annotations]] — deep dive into `[Key]`, `[ForeignKey]`, `[InverseProperty]`
- [[Validation and Concurrency Annotations]] — deep dive into `[Required]`, `[ConcurrencyCheck]`, `[Timestamp]`
- [[Conventions Overview]] — how EF Core's default rules work before annotations are applied
- [[Fluent API Overview]] — the most powerful configuration layer that overrides annotations
- [[Relationships]] — how EF Core models one-to-many, one-to-one, and many-to-many
- [[Entity Classes]] — structuring entity classes that Data Annotations decorate
