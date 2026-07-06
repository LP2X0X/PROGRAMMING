---
tags: [csharp, ef-core, conventions, fundamentals]
---

## What Are Conventions?

- **Conventions** are EF Core's built-in rules that automatically map your C# classes to database structures **without any explicit configuration**.
- This philosophy is called **convention over configuration** -- EF Core makes sensible assumptions about how your code maps to a database, so you only need to write configuration code when the defaults don't fit.
- Conventions cover table names, column names, data types, primary keys, foreign keys, relationships, nullability, and more.
- If you follow the conventions, a simple POCO class with an `Id` property and navigation properties will map correctly with **zero configuration needed**.

```csharp
// This class maps to a database table with zero configuration
public class Product
{
    public int Id { get; set; }              // PK by convention (auto-increment)
    public string Name { get; set; }         // nvarchar(max), NOT NULL
    public decimal Price { get; set; }       // decimal(18,2), NOT NULL
    public string? Description { get; set; } // nvarchar(max), nullable

    public int CategoryId { get; set; }      // FK by convention
    public Category Category { get; set; }   // Navigation property
}
```

```ad-tip
title: The 80/20 Rule
Conventions handle roughly 80% of your mapping needs. Learn them well and you'll rarely need to write explicit configuration. The remaining 20% is where Data Annotations and Fluent API come in.
```

---

## The Three Configuration Approaches

EF Core provides **three ways** to control how your model maps to the database. They have a strict **precedence order** -- when multiple approaches configure the same thing, the higher-precedence one wins.

### Precedence Hierarchy

```
Convention  <  Data Annotations  <  Fluent API
(lowest)                           (highest -- always wins)
```

| Approach | Where It Lives | When to Use |
|---|---|---|
| **Conventions** | Automatic -- built into EF Core | Default behavior. No code needed. Works when your classes follow standard patterns. |
| **Data Annotations** | Attributes on entity classes (`[Table]`, `[Required]`, `[MaxLength]`, etc.) | Simple overrides. Also works with ASP.NET model validation. |
| **Fluent API** | Code in `DbContext.OnModelCreating()` | Complex configuration, anything Data Annotations can't do, or when you want clean entity classes. |

```ad-note
title: How Precedence Works in Practice
1. EF Core applies **conventions first** -- it scans your model and makes default decisions.
2. **Data Annotations** override those defaults where present.
3. **Fluent API** overrides everything -- it always has the final say.

If you put `[MaxLength(100)]` on a property AND call `.HasMaxLength(200)` in the Fluent API, the column will be `nvarchar(200)`.
```

### When Each Approach Is the Right Choice

- **Conventions alone** -- Your POCO has `Id` or `{ClassName}Id` for its primary key. Properties are named the way you want your columns named. Navigation properties point to other entities. You don't need to write a single line of configuration.
- **Data Annotations** -- You need a simple override: `[Required]`, `[MaxLength(100)]`, `[Table("tbl_Products")]`, `[Column("product_name")]`. Especially good when the same validation also applies in ASP.NET (e.g., `[Required]` on a form model).
- **Fluent API** -- You need composite keys, query filters, advanced index configuration, many-to-many with a custom join entity, or you want to keep your entity classes completely free of EF-specific attributes.

---

## Comprehensive Convention Summary Table

This table lists **every major convention** that EF Core applies by default. This is the single most important reference for understanding what EF Core does automatically.

| Convention | What EF Core Does Automatically |
|---|---|
| **Included tables** | Each `DbSet<T>` property on your [[DbContext]] becomes a table. Types discovered through navigation properties are also included even without a `DbSet<T>`. |
| **Included columns** | All **public properties** with both a **getter and a setter** are mapped to columns. Read-only properties, private properties, static properties, and indexer properties are excluded. |
| **Table name** | Uses the `DbSet<T>` **property name** (e.g., `DbSet<Car> Cars` creates a "Cars" table). If no `DbSet<T>` exists (entity discovered via navigation), the **class name** is used. |
| **Schema** | All tables are placed in the database's **default schema** (`dbo` in SQL Server, `public` in PostgreSQL). |
| **Column name** | The **property name** becomes the column name exactly as written (e.g., `FirstName` property becomes a "FirstName" column). |
| **Column data type** | Inferred from the **.NET type** and the **database provider** (e.g., `string` maps to `nvarchar(max)` in SQL Server, `int` maps to `int`). See [[Table and Column Conventions]] for the full mapping table. |
| **Column nullability** | **Nullable types** (`string?`, `int?`) map to nullable columns. **Non-nullable types** (`string`, `int`) map to `NOT NULL` columns. Depends on whether nullable reference types (NRT) are enabled. |
| **Primary key** | A property named `Id` or `{ClassName}Id` is automatically the primary key. Case-insensitive matching. Auto-increment for `int`/`long`, database-generated for `Guid`. |
| **Foreign key** | A property named `{NavigationPropertyName}Id` or `{PrincipalTypeName}Id` becomes the foreign key column. |
| **Relationships** | Discovered from **navigation properties** -- a reference to another entity type (one-to-one or one-to-many) or a collection of entity types (one-to-many or many-to-many). |
| **Cascade delete** | **Required relationships** (non-nullable FK) default to `Cascade`. **Optional relationships** (nullable FK) default to `ClientSetNull`. |
| **Shadow properties** | If a navigation property exists but no FK property is declared in C#, EF Core creates a **shadow FK property** that exists in the model and database but not in your code. |
| **Backing fields** | If a field matches a property by naming convention (e.g., `_name` for `Name`), EF Core uses the field for reads/writes. |
| **Value generation** | `int`/`long` PKs get identity (auto-increment). `Guid` PKs get database-generated values. `string` PKs get no auto-generation. |
| **Index on FK** | EF Core automatically creates an index on foreign key columns for query performance. |

---

## When Conventions Are Enough

Conventions alone are sufficient when your model follows these patterns:

- **Primary keys** are named `Id` or `{ClassName}Id` (e.g., `CustomerId` for a `Customer` class)
- **Foreign keys** follow the `{NavigationName}Id` pattern (e.g., `CustomerId` alongside a `Customer` navigation property)
- **Table names** match your `DbSet<T>` property names
- **Column names** match your C# property names
- You're fine with the default **data types** (e.g., `string` as `nvarchar(max)`)
- **Nullability** matches your C# type declarations

```csharp
// Everything here maps by convention -- zero configuration needed
public class AppDbContext : DbContext
{
    public DbSet<Customer> Customers { get; set; }  // → "Customers" table
    public DbSet<Order> Orders { get; set; }        // → "Orders" table
}

public class Customer
{
    public int Id { get; set; }                      // PK, auto-increment
    public string Name { get; set; }                 // nvarchar(max), NOT NULL
    public string? Email { get; set; }               // nvarchar(max), nullable

    public ICollection<Order> Orders { get; set; }   // Collection nav
        = new List<Order>();
}

public class Order
{
    public int Id { get; set; }                      // PK, auto-increment
    public DateTime OrderDate { get; set; }          // datetime2, NOT NULL
    public decimal Total { get; set; }               // decimal(18,2), NOT NULL

    public int CustomerId { get; set; }              // FK by convention
    public Customer Customer { get; set; }           // Reference nav
}
```

```ad-warning
title: When Conventions Are NOT Enough
You'll need explicit configuration when:
- Your database uses **different naming** than your C# code (e.g., `snake_case` columns, prefixed table names)
- You need **composite primary keys** (conventions can't detect them)
- You want **max lengths** on string columns (convention defaults to `nvarchar(max)`)
- You need **query filters** (e.g., soft delete: `.HasQueryFilter(x => !x.IsDeleted)`)
- Your relationships are **ambiguous** (e.g., an entity has two navigation properties to the same type)
- You need to configure **inheritance mapping** (TPH, TPT, TPC)
```

---

## How EF Core Discovers Your Model

Understanding the discovery process helps you predict what conventions will produce:

1. **Start with DbSet properties** -- EF Core looks at every `DbSet<T>` on your [[DbContext]]. Each `T` becomes a known entity type.
2. **Follow navigation properties** -- For each known entity, EF Core examines properties whose types are other entity classes (or collections of them). Those referenced types become entities too, even without their own `DbSet<T>`.
3. **Apply conventions** -- For every discovered entity, EF Core applies the convention rules from the table above: find the PK, map columns, detect FKs, determine nullability, etc.
4. **Apply overrides** -- Data Annotations on the classes are processed next, overriding any convention decisions.
5. **Apply Fluent API** -- Finally, `OnModelCreating` runs, and Fluent API configuration overrides everything else.

```ad-tip
title: Debugging Model Discovery
Use `context.Model` at runtime or the `dotnet ef dbcontext info` / `dotnet ef dbcontext script` commands to see exactly what EF Core has discovered and how it's mapping your model. If something unexpected appears, check the discovery chain above.
```

---

## See Also

- [[Table and Column Conventions]] -- detailed rules for how tables and columns are named and typed
- [[Key and Relationship Conventions]] -- how EF Core discovers PKs, FKs, and relationships automatically
- [[Overriding Conventions]] -- when and how to override default behavior
- [[Entity Classes]] -- how entity classes work in EF Core
- [[DbContext]] -- where entities are registered and discovered
- [[Fluent API Configuration]] -- the most powerful configuration approach
