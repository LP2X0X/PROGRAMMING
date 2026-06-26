---
tags: [csharp, ef-core, data-annotations, keys, relationships]
---

## Overview

- This note covers the Data Annotations that configure **primary keys** and **relationships** (foreign keys, navigation resolution) in EF Core.
- All of these live in the `System.ComponentModel.DataAnnotations` or `System.ComponentModel.DataAnnotations.Schema` namespace.
- For relationship fundamentals (one-to-many, one-to-one, many-to-many), see [[Relationships]].

| Annotation | Namespace | Purpose |
| --- | --- | --- |
| `[Key]` | DataAnnotations | Mark a property as the primary key |
| `[Required]` on navigation | DataAnnotations | Make a relationship required (non-nullable FK) |
| `[ForeignKey("prop")]` | Schema | Explicitly specify which property is the FK |
| `[InverseProperty("prop")]` | Schema | Resolve ambiguous navigation properties |

---

## `[Key]` — Explicit Primary Key

- By convention, EF Core treats a property named `Id` or `{ClassName}Id` as the primary key.
- `[Key]` is needed **only** when your PK property has a non-conventional name.

### When You Need It

```csharp
public class Vehicle
{
    [Key]
    public int VehicleNumber { get; set; }  // not "Id" or "VehicleId" → needs [Key]

    public string Make { get; set; }
    public string Model { get; set; }
}
```

Without `[Key]`, EF Core would throw an exception: *"The entity type 'Vehicle' requires a primary key to be defined."*

### When You Don't Need It

```csharp
public class Vehicle
{
    public int Id { get; set; }            // convention: "Id" → auto-detected as PK
    // OR
    public int VehicleId { get; set; }     // convention: "{ClassName}Id" → auto-detected as PK
}
```

### `[Key]` with Different Data Types

```csharp
// GUID primary key
public class Session
{
    [Key]
    public Guid SessionToken { get; set; }  // non-conventional name, needs [Key]
}

// String primary key
public class Country
{
    [Key]
    [MaxLength(3)]
    public string IsoCode { get; set; }  // "USA", "GBR", etc.
}
```

```ad-warning
title: Composite Keys Cannot Use [Key] Alone in EF Core
In EF Core, you **cannot** define a composite primary key using only `[Key]` attributes. The `[Key]` + `[Column(Order = n)]` pattern from EF6 does **not work** in EF Core.

You **must** use Fluent API for composite keys:
~~~csharp
// This does NOT work in EF Core:
public class OrderItem
{
    [Key, Column(Order = 0)]
    public int OrderId { get; set; }    // ← EF Core ignores Column(Order) for composite keys

    [Key, Column(Order = 1)]
    public int ProductId { get; set; }  // ← Error: multiple [Key] properties found
}

// This DOES work — use Fluent API:
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<OrderItem>()
        .HasKey(oi => new { oi.OrderId, oi.ProductId });
}
~~~

This is one of the most common EF6-to-EF Core migration gotchas.
```

### `[Key]` with `[DatabaseGenerated]`

- By default, `int`/`long` PKs get `Identity` generation (auto-increment).
- If you want to supply the key yourself, pair `[Key]` with `[DatabaseGenerated(None)]`.

```csharp
public class Country
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.None)]
    public int NumericCode { get; set; }  // ISO 3166 numeric code — you set it, DB doesn't auto-generate

    public string Name { get; set; }
}
```

See [[Table and Column Annotations]] for full details on `[DatabaseGenerated]`.

---

## `[Required]` on Navigation Properties — Required Relationships

- Placing `[Required]` on a **reference navigation property** makes the relationship required — the FK column becomes `NOT NULL`.
- This is separate from `[Required]` on scalar properties (which is about column nullability and validation — see [[Validation and Concurrency Annotations]]).

### Without `[Required]` (Optional Relationship)

```csharp
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }

    public int? CustomerId { get; set; }       // nullable FK → optional relationship
    public Customer Customer { get; set; }
}
```

The FK `CustomerId` is nullable — an `Order` can exist without a `Customer`.

### With `[Required]` (Required Relationship)

```csharp
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }

    public int CustomerId { get; set; }        // non-nullable FK

    [Required]
    public Customer Customer { get; set; }     // relationship is required
}
```

- The FK `CustomerId` becomes `NOT NULL`.
- Attempting to save an `Order` without a `Customer` will throw a database exception.

```ad-note
With **nullable reference types** enabled (which is the default in .NET 6+), a non-nullable navigation property (`public Customer Customer { get; set; }` without the `?`) already signals to EF Core that the relationship is required. In that context, `[Required]` is redundant for EF Core — but it still adds ASP.NET validation if the entity is used as a model.
```

### Resulting SQL Comparison

```sql
-- Optional relationship (no [Required]):
CustomerId int NULL,
FOREIGN KEY (CustomerId) REFERENCES Customers(Id) ON DELETE SET NULL

-- Required relationship (with [Required]):
CustomerId int NOT NULL,
FOREIGN KEY (CustomerId) REFERENCES Customers(Id) ON DELETE CASCADE
```

```ad-tip
**Delete behavior follows requirement**: Required relationships default to `CASCADE` delete (delete the parent → delete the children). Optional relationships default to `SET NULL` (delete the parent → set FK to NULL). You can override this with Fluent API `.OnDelete(DeleteBehavior.X)`.
```

---

## `[ForeignKey]` — Explicit Foreign Key Mapping

- By convention, EF Core matches a FK property to a navigation property using the naming pattern `{NavigationPropertyName}Id`.
- `[ForeignKey]` is needed when the FK property name **doesn't follow** this convention.

### Two Placement Options

You can place `[ForeignKey]` on either the FK property or the navigation property. Both are equivalent.

#### Option 1: On the FK Property (Naming the Navigation)

```csharp
public class Order
{
    public int Id { get; set; }

    [ForeignKey("Customer")]              // "Customer" = name of the navigation property
    public int CustId { get; set; }       // doesn't follow {NavigationName}Id pattern

    public Customer Customer { get; set; }
}
```

- `CustId` doesn't match `CustomerId`, so EF Core needs `[ForeignKey]` to connect it to the `Customer` navigation.

#### Option 2: On the Navigation Property (Naming the FK)

```csharp
public class Order
{
    public int Id { get; set; }

    public int CustId { get; set; }

    [ForeignKey("CustId")]                // "CustId" = name of the FK property
    public Customer Customer { get; set; }
}
```

Both options produce the exact same database schema. Choose whichever reads more clearly in your codebase.

```ad-tip
**Convention**: Most teams pick one style and stick with it. Option 2 (annotation on the navigation) is slightly more common because the navigation property is the "relationship" and the FK is its "implementation detail."
```

### `[ForeignKey]` When the FK Property is Not in the Entity

- If you don't have an explicit FK property, EF Core creates a **shadow property**.
- `[ForeignKey]` can point to a shadow property name, but this is unusual. It's better to include the FK property explicitly.

```csharp
// No explicit FK property — EF Core creates a shadow property named "CustomerId"
public class Order
{
    public int Id { get; set; }
    public Customer Customer { get; set; }
}
// Works by convention, but you can't do db.Orders.Where(o => o.CustomerId == 5)
// because CustomerId doesn't exist in C# — it's only in the DB.
```

```ad-warning
title: Always Include the FK Property Explicitly
Shadow foreign keys work, but they force you to use navigation properties for all queries involving the FK. Including the FK property lets you filter efficiently:
~~~csharp
// With explicit FK property:
var orders = db.Orders.Where(o => o.CustomerId == 5).ToList();  // simple, no JOIN

// Without (shadow FK):
var orders = db.Orders.Where(o => o.Customer.Id == 5).ToList(); // EF may generate a JOIN
~~~
```

### `[ForeignKey]` with Composite Foreign Keys

- Composite FKs (multi-column) **cannot** be fully configured with `[ForeignKey]` alone in EF Core.
- You need Fluent API for composite FK definitions:

```csharp
// Fluent API in OnModelCreating:
modelBuilder.Entity<OrderItem>()
    .HasOne(oi => oi.Order)
    .WithMany(o => o.Items)
    .HasForeignKey(oi => new { oi.OrderId, oi.OrderYear });  // composite FK
```

---

## `[InverseProperty]` — Resolve Ambiguous Navigations

- When an entity has **multiple navigation properties** pointing to the **same related type**, EF Core can't tell which navigation on one side pairs with which navigation on the other.
- `[InverseProperty]` explicitly tells EF Core which navigations are paired.

### The Problem — Ambiguous Navigations

```csharp
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }

    // Two collections pointing to the same type (BlogPost)
    public ICollection<BlogPost> AuthoredPosts { get; set; }
    public ICollection<BlogPost> EditedPosts { get; set; }
}

public class BlogPost
{
    public int Id { get; set; }
    public string Title { get; set; }

    // Two reference navigations pointing to the same type (Employee)
    public int AuthorId { get; set; }
    public Employee Author { get; set; }

    public int? EditorId { get; set; }
    public Employee Editor { get; set; }
}
```

Without `[InverseProperty]`, EF Core throws:

> *Unable to determine the relationship represented by navigation 'Employee.AuthoredPosts'... Manually configure the relationship, or ignore this property using '[NotMapped]'.*

### The Solution — `[InverseProperty]`

```csharp
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }

    [InverseProperty("Author")]           // matches BlogPost.Author navigation
    public ICollection<BlogPost> AuthoredPosts { get; set; }

    [InverseProperty("Editor")]           // matches BlogPost.Editor navigation
    public ICollection<BlogPost> EditedPosts { get; set; }
}

public class BlogPost
{
    public int Id { get; set; }
    public string Title { get; set; }

    public int AuthorId { get; set; }
    public Employee Author { get; set; }

    public int? EditorId { get; set; }
    public Employee Editor { get; set; }
}
```

- `[InverseProperty("Author")]` says: "The `AuthoredPosts` collection is the inverse of the `Author` navigation in `BlogPost`."
- `[InverseProperty("Editor")]` says: "The `EditedPosts` collection is the inverse of the `Editor` navigation in `BlogPost`."

### Resulting SQL

```sql
CREATE TABLE Employees (
    Id   int           IDENTITY(1,1) PRIMARY KEY,
    Name nvarchar(max) NOT NULL
);

CREATE TABLE BlogPosts (
    Id       int           IDENTITY(1,1) PRIMARY KEY,
    Title    nvarchar(max) NOT NULL,
    AuthorId int           NOT NULL,
    EditorId int           NULL,
    FOREIGN KEY (AuthorId) REFERENCES Employees(Id) ON DELETE CASCADE,
    FOREIGN KEY (EditorId) REFERENCES Employees(Id) ON DELETE NO ACTION
);
```

```ad-warning
title: SQL Server Multiple Cascade Paths
SQL Server does **not allow** multiple FK relationships to the same table with `CASCADE` delete on both, because it can create circular or ambiguous cascade paths. EF Core automatically sets the second relationship to `NO ACTION` to avoid this. If you need cascade on both, you must handle deletion logic manually (e.g., triggers or application code).
```

### Another Common Scenario — Self-Referencing Entity

```csharp
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }

    public int? ManagerId { get; set; }

    [ForeignKey("ManagerId")]
    [InverseProperty("DirectReports")]
    public Employee Manager { get; set; }

    [InverseProperty("Manager")]
    public ICollection<Employee> DirectReports { get; set; } = new List<Employee>();
}
```

- This models a manager-subordinate hierarchy within a single table.
- `[InverseProperty]` resolves the ambiguity because both navigations reference the same type (`Employee`).

---

## Complete Relationship Example — Combining All Annotations

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

public class Department
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.None)]
    public int DepartmentCode { get; set; }       // natural key, manually assigned

    [Required]
    [MaxLength(100)]
    public string Name { get; set; }

    [InverseProperty("Department")]
    public ICollection<Employee> Employees { get; set; } = new List<Employee>();

    [InverseProperty("ManagedDepartment")]
    public Employee DepartmentHead { get; set; }
}

public class Employee
{
    public int Id { get; set; }

    [Required]
    [MaxLength(150)]
    public string Name { get; set; }

    // Required relationship to Department
    [ForeignKey("Department")]
    public int DeptCode { get; set; }             // non-conventional FK name → needs [ForeignKey]

    [Required]
    public Department Department { get; set; }

    // Optional self-referencing relationship
    public int? ManagerId { get; set; }

    [ForeignKey("ManagerId")]
    public Employee Manager { get; set; }

    [InverseProperty("Manager")]
    public ICollection<Employee> DirectReports { get; set; } = new List<Employee>();

    // Optional 1:1 — this employee may head a department
    public int? ManagedDepartmentCode { get; set; }

    [ForeignKey("ManagedDepartmentCode")]
    public Department ManagedDepartment { get; set; }
}
```

### Resulting SQL (SQL Server)

```sql
CREATE TABLE Departments (
    DepartmentCode int          PRIMARY KEY,         -- no IDENTITY (DatabaseGenerated.None)
    Name           nvarchar(100) NOT NULL
);

CREATE TABLE Employees (
    Id                      int           IDENTITY(1,1) PRIMARY KEY,
    Name                    nvarchar(150) NOT NULL,
    DeptCode                int           NOT NULL,  -- required FK to Department
    ManagerId               int           NULL,      -- optional self-referencing FK
    ManagedDepartmentCode   int           NULL,      -- optional FK for 1:1 relationship
    FOREIGN KEY (DeptCode) REFERENCES Departments(DepartmentCode),
    FOREIGN KEY (ManagerId) REFERENCES Employees(Id),
    FOREIGN KEY (ManagedDepartmentCode) REFERENCES Departments(DepartmentCode)
);
```

---

## Summary — When to Use Each Annotation

| I Want To... | Use This |
| --- | --- |
| Mark a non-conventional PK | `[Key]` |
| Define a composite PK | Fluent API (not possible with annotations alone in EF Core) |
| Make a relationship required | `[Required]` on the navigation |
| Map a non-conventional FK property | `[ForeignKey("name")]` |
| Resolve ambiguous navigations to same type | `[InverseProperty("name")]` |
| Model self-referencing hierarchy | `[ForeignKey]` + `[InverseProperty]` |
| Suppress auto-generated key values | `[Key]` + `[DatabaseGenerated(None)]` |

---

## Cross-References

- [[Data Annotations Overview]] — full annotation reference and precedence rules
- [[Relationships]] — in-depth coverage of one-to-many, one-to-one, and many-to-many
- [[Table and Column Annotations]] — `[Table]`, `[Column]`, `[NotMapped]`, `[DatabaseGenerated]`
- [[Validation and Concurrency Annotations]] — `[Required]` for scalar properties, `[ConcurrencyCheck]`, `[Timestamp]`
- [[Conventions Overview]] — the naming conventions that `[Key]` and `[ForeignKey]` override
- [[Fluent API Overview]] — Fluent equivalents like `.HasKey()`, `.HasOne().WithMany()`, `.HasForeignKey()`
