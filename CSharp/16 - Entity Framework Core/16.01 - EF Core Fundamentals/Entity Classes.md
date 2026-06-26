---
tags: [csharp, ef-core, entities, fundamentals]
---

## What Are Entity Classes

- An **entity class** is a plain C# class (POCO — Plain Old CLR Object) that maps to a **database table**.
- Each **instance** of the class maps to a **row** in that table.
- Each **property** maps to a **column**.
- EF Core discovers entity classes through the `DbSet<T>` properties on your [[DbContext]].

```csharp
// This class maps to a "Customers" table
public class Customer
{
    public int Id { get; set; }         // PK column
    public string Name { get; set; }    // Name column
    public string Email { get; set; }   // Email column
}
```

---

## EF Core Conventions (Convention Over Configuration)

EF Core follows **conventions** so you don't have to configure everything manually. Learn the conventions first — they cover 80% of cases.

### Table Name Convention

- The `DbSet<T>` **property name** becomes the table name.
- `public DbSet<Customer> Customers { get; set; }` maps to the **"Customers"** table.
- If there is no `DbSet<T>` property (the entity is discovered through a navigation property), the **class name** is used.

### Primary Key Convention

- A property named `Id` or `{ClassName}Id` is automatically treated as the **primary key**.
- Both `Id` and `CustomerId` work for a class named `Customer`.
- If the PK is `int`, `long`, or `Guid`, EF Core also configures **auto-generation** (identity column for int/long, `newsequentialid()` for Guid in SQL Server).

```csharp
public class Customer
{
    public int Id { get; set; }            // ✅ PK by convention (named "Id")
}

public class Order
{
    public int OrderId { get; set; }       // ✅ PK by convention (named "{ClassName}Id")
}
```

### Foreign Key Convention

- A property named `{NavigationPropertyName}Id` or `{RelatedClassName}Id` is treated as a **foreign key**.
- See [[Relationships]] for full details.

```csharp
public class Order
{
    public int OrderId { get; set; }

    public int CustomerId { get; set; }    // FK by convention
    public Customer Customer { get; set; } // Navigation property
}
```

### Navigation Property Convention

- A property whose type is another entity class (or a collection of entities) is a **navigation property**.
- EF Core uses navigation properties to infer [[Relationships|relationships]] between tables.

### Nullable Convention

- **Nullable** reference types (`string?`) or nullable value types (`int?`) map to **nullable columns**.
- Non-nullable types map to **NOT NULL** columns (when nullable reference types are enabled in the project).

---

## Three Ways to Configure Entities

When conventions aren't enough, you configure entities explicitly. EF Core supports three approaches, each with increasing power:

1. **[[Conventions Overview|Conventions]]** — automatic defaults (e.g., `Id` → PK, `DbSet` property name → table name)
2. **[[Data Annotations Overview|Data Annotations]]** — attributes on entity classes (`[Required]`, `[MaxLength]`, `[Table]`)
3. **[[Fluent API Overview|Fluent API]]** — code in `OnModelCreating()` with full configuration power

```ad-note
title: Precedence Rule
Convention < Data Annotations < Fluent API. If both an annotation and Fluent API configure the same thing, **Fluent API wins**.
```

See the dedicated folders for each approach: [[16.04 - Conventions]], [[16.05 - Data Annotations]], [[16.06 - Fluent API]].

---

## Full Entity Example

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

[Table("Products")]
public class Product
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }

    [Required]
    [MaxLength(150)]
    public string Name { get; set; }

    [MaxLength(1000)]
    public string? Description { get; set; }     // nullable column

    [Column(TypeName = "decimal(10,2)")]         // explicit SQL type
    public decimal Price { get; set; }

    public bool IsActive { get; set; } = true;   // default value

    [NotMapped]
    public string Summary => $"{Name} - {Price:C}";  // not in DB

    // Navigation property — this product belongs to a category
    public int CategoryId { get; set; }           // FK by convention
    public Category Category { get; set; }        // Navigation
}

public class Category
{
    public int Id { get; set; }

    [Required]
    [MaxLength(100)]
    public string Name { get; set; }

    // Navigation — a category has many products
    public ICollection<Product> Products { get; set; } = new List<Product>();
}
```

```ad-warning
**Entity classes must have a parameterless constructor** (or EF Core must be able to use the one you provide). EF Core creates instances using `Activator.CreateInstance` or compiled expressions — if the only constructor requires parameters EF Core can't find, materialization will fail at runtime.
```

---

## Value Objects and Owned Types (Brief Mention)

- For properties that are not standalone entities but structured values (e.g., an `Address` with Street, City, Zip), EF Core supports **Owned Types**.
- Owned types store their columns in the **parent's table** (no separate table).

```csharp
[Owned]
public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public string ZipCode { get; set; }
}

public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public Address ShippingAddress { get; set; }  // columns: ShippingAddress_Street, etc.
}
```

- This is an advanced topic — the key thing to know for fundamentals is that **not every class needs its own table**.

---

## See Also

- [[DbContext]] — where entities are registered and configured
- [[Relationships]] — navigation properties, foreign keys, and relationship types
- [[EF Core Overview]] — the big picture of how entities fit into EF Core
- [[Foreign Keys and Relationships]] — the SQL-side view of relationships
