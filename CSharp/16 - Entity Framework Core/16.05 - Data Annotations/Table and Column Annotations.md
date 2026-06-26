---
tags: [csharp, ef-core, data-annotations, tables, columns]
---

## Overview

- The annotations in this note control **how entity classes and properties map to database tables and columns**.
- They all live in the `System.ComponentModel.DataAnnotations.Schema` namespace (except `[MaxLength]` and `[StringLength]`, which are in `System.ComponentModel.DataAnnotations`).
- These annotations override EF Core's [[Conventions Overview|default naming and type conventions]] — they sit in the middle of the precedence chain (Convention < **Data Annotations** < [[Fluent API Overview|Fluent API]]).

```csharp
using System.ComponentModel.DataAnnotations;        // [MaxLength], [StringLength]
using System.ComponentModel.DataAnnotations.Schema;  // [Table], [Column], [NotMapped], [DatabaseGenerated]
```

---

## `[Table]` — Override Table Name

- By convention, EF Core names the table after the `DbSet<T>` property name in your DbContext.
- `[Table("name")]` overrides that table name.
- `[Table("name", Schema = "schema")]` also sets the database schema.

### Basic Usage

```csharp
[Table("tbl_Vehicles")]
public class Car
{
    public int Id { get; set; }
    public string Make { get; set; }
    public string Model { get; set; }
}
```

**Convention table name**: `Cars` (from `DbSet<Car> Cars`)
**With annotation**: `tbl_Vehicles`

### With Schema

```csharp
[Table("Vehicles", Schema = "inventory")]
public class Car
{
    public int Id { get; set; }
    public string Make { get; set; }
}
```

**Resulting SQL (SQL Server)**:

```sql
CREATE TABLE inventory.Vehicles (
    Id    int           IDENTITY(1,1) PRIMARY KEY,
    Make  nvarchar(max) NULL,
    Model nvarchar(max) NULL
);
```

```ad-note
If you don't specify `Schema`, the table goes into the database's **default schema** (usually `dbo` on SQL Server). You can change the default schema for all entities via Fluent API: `modelBuilder.HasDefaultSchema("myapp")`.
```

```ad-tip
**When to use `[Table]`**: Use it when your DBA requires specific table naming conventions (e.g., `tbl_` prefix, snake_case), or when you're mapping to an existing database where table names don't match your C# class names. If your class name and table name match (or you're happy with the `DbSet` property name), you don't need it.
```

---

## `[Column]` — Override Column Name, Type, and Order

- `[Column]` controls how a property maps to its database column.
- It has three configurable aspects: **Name**, **TypeName**, and **Order**.

### Override Column Name

```csharp
public class Customer
{
    public int Id { get; set; }

    [Column("email_address")]      // column name in DB
    public string Email { get; set; }

    [Column("phone_number")]
    public string Phone { get; set; }
}
```

**Convention**: columns would be `Email` and `Phone`.
**With annotation**: columns are `email_address` and `phone_number`.

```sql
CREATE TABLE Customers (
    Id            int           IDENTITY(1,1) PRIMARY KEY,
    email_address nvarchar(max) NULL,
    phone_number  nvarchar(max) NULL
);
```

### Override Column Type with `TypeName`

- `TypeName` lets you specify the **exact SQL data type** for the column.
- This is critical for `decimal` properties, where the default precision may not match your needs.

```csharp
public class Product
{
    public int Id { get; set; }

    [Column(TypeName = "decimal(10,2)")]    // 10 digits total, 2 after decimal
    public decimal Price { get; set; }

    [Column(TypeName = "varchar(50)")]      // varchar instead of nvarchar (no Unicode)
    public string Sku { get; set; }

    [Column(TypeName = "date")]             // date only, no time component
    public DateTime ManufacturedDate { get; set; }

    [Column(TypeName = "text")]             // legacy text type (not recommended, but useful for existing DBs)
    public string LongDescription { get; set; }
}
```

```sql
CREATE TABLE Products (
    Id               int           IDENTITY(1,1) PRIMARY KEY,
    Price            decimal(10,2) NOT NULL,
    Sku              varchar(50)   NULL,
    ManufacturedDate date          NOT NULL,
    LongDescription  text          NULL
);
```

```ad-warning
title: Decimal Precision Matters
On SQL Server, the default mapping for `decimal` is `decimal(18,2)`. If you need different precision (e.g., currency with 4 decimal places, or scientific values with 6), **you must specify it** with `[Column(TypeName = "decimal(p,s)")]`. Getting this wrong can silently truncate values.

Example: storing `123.456789` in a `decimal(10,2)` column saves `123.46` — the extra digits are silently rounded.
```

### Combining Name and TypeName

```csharp
[Column("unit_price", TypeName = "decimal(10,4)")]
public decimal UnitPrice { get; set; }
// → column "unit_price" with type decimal(10,4)
```

### Column Order

- `[Column(Order = n)]` provides a **hint** about column position in the table.
- In EF Core, this is largely cosmetic — the provider decides the actual column order in most cases.
- It was more meaningful in EF6 for defining composite key column order.

```csharp
[Column(Order = 0)]
public int Id { get; set; }

[Column(Order = 1)]
public string Name { get; set; }

[Column(Order = 2)]
public string Email { get; set; }
```

```ad-note
EF Core 7+ does respect `[Column(Order = n)]` for column ordering in the generated CREATE TABLE statement. Earlier versions largely ignore it. Even so, column order in a relational database is cosmetic — SQL queries are not affected by physical column order.
```

---

## `[NotMapped]` — Exclude from Database

- `[NotMapped]` tells EF Core to **completely ignore** a property or class — it won't create a column (or table) for it.
- Use it for computed properties, display helpers, or any data that lives only in C# memory.

### Exclude a Property

```csharp
public class Employee
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }

    [NotMapped]
    public string FullName => $"{FirstName} {LastName}";  // computed, not stored

    [NotMapped]
    public int Age { get; set; }  // maybe calculated from birthdate at runtime
}
```

`FullName` and `Age` will **not** appear as columns in the `Employees` table.

### Exclude an Entire Class

```csharp
[NotMapped]
public class AuditMetadata
{
    public string CreatedBy { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

- If `AuditMetadata` is referenced as a property type in another entity, EF Core won't try to map it.
- Useful for DTOs, view models, or helper classes that accidentally end up in the entity assembly.

```ad-tip
**Common use case**: You have a property that exists purely for the UI layer (e.g., `IsSelected`, `DisplayColor`, `ValidationMessage`). Mark it `[NotMapped]` so EF Core doesn't try to create a column for it.
```

```ad-warning
title: NotMapped vs Private Setters
`[NotMapped]` is for properties you **never** want in the database. If you want a property that EF Core can read/write but isn't publicly settable, use a private setter instead: `public string Secret { get; private set; }` — EF Core can still map this.
```

---

## `[DatabaseGenerated]` — Control Value Generation

- `[DatabaseGenerated(DatabaseGeneratedOption.X)]` tells EF Core **whether the database generates a value** for this column and **when**.
- Three options:

| Option | Behavior | Typical Use |
| --- | --- | --- |
| `Identity` | Database generates value on **INSERT** only | Auto-increment PKs, identity columns |
| `Computed` | Database generates value on **INSERT and UPDATE** | Computed columns, triggers |
| `None` | Database does **not** generate a value — you must provide it | Natural keys, GUIDs generated in app code |

### `DatabaseGeneratedOption.Identity`

```csharp
public class Order
{
    // This is the default for int/long PKs — you usually don't need to write it.
    // But it's explicit here for clarity.
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }

    // A non-PK identity column (e.g., an order number generated by the DB)
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int OrderNumber { get; set; }
}
```

- EF Core will **not include** this column in INSERT statements — it lets the database assign the value.
- After `SaveChanges()`, EF Core reads back the generated value.

### `DatabaseGeneratedOption.Computed`

```csharp
public class Product
{
    public int Id { get; set; }
    public decimal Price { get; set; }
    public decimal TaxRate { get; set; }

    [DatabaseGenerated(DatabaseGeneratedOption.Computed)]
    public decimal TotalPrice { get; set; }
    // Assumes a computed column in SQL: ALTER TABLE Products ADD TotalPrice AS (Price * (1 + TaxRate))
}
```

- EF Core excludes this column from both INSERT and UPDATE statements.
- It reads the value back from the database after every save operation.

```ad-warning
title: Computed Columns Are Read-Only in EF Core
If you mark a property as `DatabaseGeneratedOption.Computed`, EF Core will **never** send a value for it to the database. Attempting to set it in C# and save will have no effect — the database always computes it. Make sure the column actually has a computation defined on the database side.
```

### `DatabaseGeneratedOption.None`

```csharp
public class Country
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.None)]
    public int Id { get; set; }  // you must assign this yourself (e.g., ISO country code number)

    public string Name { get; set; }
    public string IsoCode { get; set; }
}
```

- EF Core will **include** this column in INSERT statements — it expects you to provide the value.
- Use this for natural/meaningful keys, or when you generate IDs in application code (e.g., `Guid.NewGuid()`).

```ad-tip
**GUID primary keys**: If you use `Guid` as your PK type, EF Core defaults to `DatabaseGeneratedOption.None` and generates the GUID client-side. This is usually what you want. If your database has a `DEFAULT NEWSEQUENTIALID()`, set it to `Identity` so EF Core lets the DB handle it.
```

---

## `[MaxLength]` vs `[StringLength]` — Setting Column Length

- Both annotations set the **maximum length** of a string (or byte array) column.
- The key difference is their **validation behavior**:

| Annotation | Namespace | EF Core Effect | ASP.NET Validation | Works On |
| --- | --- | --- | --- | --- |
| `[MaxLength(n)]` | DataAnnotations | `nvarchar(n)` column | **No** | `string`, `byte[]` |
| `[StringLength(n)]` | DataAnnotations | `nvarchar(n)` column | **Yes** — max length validated | `string` only |

### `[MaxLength]` — Database Only

```csharp
public class Product
{
    public int Id { get; set; }

    [MaxLength(100)]                     // nvarchar(100) in DB, no validation
    public string Sku { get; set; }

    [MaxLength(1024)]                    // varbinary(1024) in DB
    public byte[] Thumbnail { get; set; }
}
```

### `[StringLength]` — Database + Validation

```csharp
public class Product
{
    public int Id { get; set; }

    [StringLength(200, MinimumLength = 3, ErrorMessage = "Name must be 3-200 characters")]
    public string Name { get; set; }
    // → nvarchar(200) in DB
    // → ASP.NET validates: min 3, max 200 characters
}
```

```ad-tip
**Which to use?**
- If the property is only used by EF Core (no web forms/API input): use `[MaxLength]`.
- If the property is also bound to user input in ASP.NET (forms, API models): use `[StringLength]` to get validation for free.
- You can use both on the same property, but it's redundant — `[StringLength]` already sets the DB column length.
```

```ad-warning
title: Default Without Either Annotation
If you don't specify `[MaxLength]` or `[StringLength]` on a `string` property, EF Core maps it to `nvarchar(max)` (SQL Server) — which means **unlimited length**. This is often wasteful. For columns where you know the practical limit (names, emails, SKUs), always set a max length.
```

---

## Full Example — All Table and Column Annotations Together

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

[Table("tbl_Products", Schema = "catalog")]
public class Product
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }

    [Required]
    [MaxLength(200)]
    [Column("product_name")]                           // custom column name
    public string Name { get; set; }

    [Column("product_desc")]
    [StringLength(4000)]                               // nvarchar(4000)
    public string Description { get; set; }

    [Column(TypeName = "decimal(10,2)")]               // exact SQL type
    public decimal Price { get; set; }

    [Column(TypeName = "varchar(50)")]                 // non-Unicode
    [MaxLength(50)]
    public string Sku { get; set; }

    [DatabaseGenerated(DatabaseGeneratedOption.Computed)]
    public decimal PriceWithTax { get; set; }          // computed column in DB

    [NotMapped]
    public string DisplayLabel => $"{Sku}: {Name}";    // not stored in DB

    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public DateTime CreatedDate { get; set; }          // DEFAULT GETDATE() on INSERT
}
```

### Resulting SQL (SQL Server)

```sql
CREATE TABLE catalog.tbl_Products (
    Id            int            IDENTITY(1,1) PRIMARY KEY,
    product_name  nvarchar(200)  NOT NULL,
    product_desc  nvarchar(4000) NULL,
    Price         decimal(10,2)  NOT NULL,
    Sku           varchar(50)    NULL,
    PriceWithTax  AS (Price * 1.1),     -- computed column (defined separately)
    CreatedDate   datetime2      NOT NULL DEFAULT GETDATE()
);
-- DisplayLabel is NOT in the table ([NotMapped])
```

---

## Summary Table — When to Use Each Annotation

| I Want To... | Use This Annotation |
| --- | --- |
| Rename the table | `[Table("name")]` |
| Set the database schema | `[Table("name", Schema = "x")]` |
| Rename a column | `[Column("name")]` |
| Set exact SQL type | `[Column(TypeName = "type")]` |
| Limit string/byte length (DB only) | `[MaxLength(n)]` |
| Limit string length (DB + validation) | `[StringLength(n)]` |
| Exclude property from DB | `[NotMapped]` |
| Exclude entire class from DB | `[NotMapped]` on class |
| Auto-increment column | `[DatabaseGenerated(Identity)]` |
| Computed column | `[DatabaseGenerated(Computed)]` |
| No auto-generation (manual key) | `[DatabaseGenerated(None)]` |

---

## Cross-References

- [[Data Annotations Overview]] — big-picture view of all annotations and when to use them
- [[Key and Relationship Annotations]] — `[Key]`, `[ForeignKey]`, `[InverseProperty]`
- [[Validation and Concurrency Annotations]] — `[Required]`, `[ConcurrencyCheck]`, `[Timestamp]`
- [[Conventions Overview]] — the default naming/type rules these annotations override
- [[Fluent API Overview]] — Fluent API equivalents like `.ToTable()`, `.HasColumnName()`, `.HasColumnType()`
