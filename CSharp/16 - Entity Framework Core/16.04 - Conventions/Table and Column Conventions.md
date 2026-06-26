---
tags: [csharp, ef-core, conventions, tables, columns]
---

## Table Discovery

EF Core discovers which classes become database tables through **two mechanisms**:

### 1. DbSet Properties on DbContext

Every `DbSet<T>` property on your [[DbContext]] class registers `T` as an entity type. The entity gets its own table.

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Customer> Customers { get; set; }   // ✅ "Customers" table
    public DbSet<Order> Orders { get; set; }         // ✅ "Orders" table
    // Product has no DbSet -- but may still get a table (see below)
}
```

### 2. Discovery Through Navigation Properties

If an entity class is referenced by a **navigation property** on an already-discovered entity, EF Core includes it automatically -- even without its own `DbSet<T>`.

```csharp
public class Order
{
    public int Id { get; set; }
    public int ProductId { get; set; }
    public Product Product { get; set; }   // ← EF Core discovers Product through this
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    // No DbSet<Product> on DbContext, but Product STILL gets a table
    // because Order.Product navigation property references it
}
```

```ad-note
title: Discovery Is Recursive
If `Product` has a navigation to `Supplier`, and `Supplier` has a navigation to `Country`, all three get discovered even if only `Order` has a `DbSet`. The discovery walks the entire navigation graph.
```

### What Gets Excluded

- Classes with **no `DbSet`** AND **no navigation property** pointing to them are **not** included.
- Properties marked with `[NotMapped]` or `.Ignore()` in Fluent API are excluded from discovery.
- **Owned types** (marked with `[Owned]` or `.OwnsOne()`/`.OwnsMany()`) don't get their own table by default -- their columns are stored in the owner's table.

---

## Table Naming Convention

The table name is determined by this priority:

1. **If a `DbSet<T>` exists** -- the **property name** of the `DbSet<T>` becomes the table name.
2. **If no `DbSet<T>` exists** (discovered via navigation) -- the **class name** itself becomes the table name.

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Car> Cars { get; set; }            // Table name: "Cars" (from property name)
    public DbSet<Car> Vehicles { get; set; }        // Table name: "Vehicles" (property name, NOT class name)
    // If Make is discovered via Car.Make navigation → Table name: "Make" (class name)
}
```

```ad-warning
title: Common Pitfall -- Pluralization
EF Core does **NOT** automatically pluralize or singularize table names. The table name is **exactly** what you name your `DbSet` property. If you write `DbSet<Car> Car`, the table name is "Car" (singular). If you write `DbSet<Car> Cars`, it's "Cars" (plural). Most developers use plural `DbSet` names by convention, but EF Core doesn't enforce this.
```

### Table Name Examples

| DbContext Property | Discovered Via Navigation | Resulting Table Name |
|---|---|---|
| `DbSet<Customer> Customers` | -- | "Customers" |
| `DbSet<Order> Orders` | -- | "Orders" |
| `DbSet<Order> CustomerOrders` | -- | "CustomerOrders" |
| No DbSet | `Customer.Address` navigation | "Address" |
| No DbSet | `Order.OrderItem` navigation | "OrderItem" |

---

## Schema Convention

- All tables are created in the **database's default schema**.
- For **SQL Server**: the default schema is `dbo` (e.g., `dbo.Customers`).
- For **PostgreSQL**: the default schema is `public`.
- EF Core does not assign a specific schema by convention -- it lets the database provider use whatever its default is.
- To override, use `[Table("Customers", Schema = "sales")]` or `.ToTable("Customers", "sales")` in Fluent API.

---

## Column Inclusion Rules

EF Core decides which properties become columns based on these rules:

### Included (Becomes a Column)

- **Public** properties
- Must have **both a getter AND a setter** (the setter can be private: `public string Name { get; private set; }` still works)
- Can be any supported .NET type (primitives, strings, enums, value types, etc.)

### Excluded (NOT a Column)

| Excluded Type | Example | Why |
|---|---|---|
| Read-only properties | `public string FullName => $"{First} {Last}";` | No setter -- EF Core can't write values back |
| Static properties | `public static int Counter { get; set; }` | Belongs to the type, not to an instance/row |
| Properties with no getter | (rare) | EF Core can't read the value |
| Indexer properties | `public string this[string key]` | Not a simple property |
| Properties marked `[NotMapped]` | `[NotMapped] public string DisplayName { get; set; }` | Explicitly excluded |

```csharp
public class Employee
{
    public int Id { get; set; }                    // ✅ Column -- public get + set
    public string Name { get; set; }               // ✅ Column
    public string Email { get; private set; }      // ✅ Column -- private setter is fine
    
    public string FullName => $"{Name}";           // ❌ NOT a column -- no setter (computed)
    public static int TotalCount { get; set; }     // ❌ NOT a column -- static
    
    [NotMapped]
    public int TempCalculation { get; set; }       // ❌ NOT a column -- explicitly excluded
}
```

```ad-tip
title: Private Setters Are Useful
`public string Name { get; private set; }` still becomes a column. EF Core can set it via reflection during materialization. This is useful for enforcing domain rules -- external code can't directly set the value, but EF Core can still populate it from the database.
```

---

## Column Naming Convention

The **property name** becomes the **column name** exactly as written. There is no transformation applied.

| Property Name | Column Name |
|---|---|
| `FirstName` | "FirstName" |
| `EmailAddress` | "EmailAddress" |
| `IsActive` | "IsActive" |
| `orderDate` | "orderDate" (preserves casing) |

```ad-warning
title: No Automatic snake_case or Other Transformations
EF Core does **not** convert `PascalCase` properties to `snake_case` columns or apply any naming transformation. If your database uses `snake_case` (common in PostgreSQL), you must explicitly configure column names using `[Column("first_name")]` or `.HasColumnName("first_name")`, or use a bulk naming convention (e.g., the `EFCore.NamingConventions` NuGet package).
```

---

## Column Data Type Mapping

EF Core infers the SQL column type from your .NET property type. The exact mapping depends on the **database provider**. Below is the complete mapping for **SQL Server** (the most commonly used provider):

### .NET to SQL Server Type Mapping

| .NET Type | SQL Server Type | Notes |
|---|---|---|
| `string` | `nvarchar(max)` | Unicode, unlimited length. Use `[MaxLength]` to constrain. |
| `char` | `nchar(1)` | Single character |
| `int` | `int` | 32-bit integer |
| `long` (`Int64`) | `bigint` | 64-bit integer |
| `short` (`Int16`) | `smallint` | 16-bit integer |
| `byte` | `tinyint` | 8-bit unsigned integer |
| `bool` | `bit` | 0 or 1 |
| `decimal` | `decimal(18,2)` | 18 digits total, 2 decimal places |
| `double` | `float` | 64-bit floating point |
| `float` (`Single`) | `real` | 32-bit floating point |
| `DateTime` | `datetime2(7)` | High-precision date/time |
| `DateTimeOffset` | `datetimeoffset(7)` | Date/time with timezone offset |
| `DateOnly` | `date` | Date without time (.NET 6+) |
| `TimeOnly` | `time(7)` | Time without date (.NET 6+) |
| `TimeSpan` | `time(7)` | Duration stored as time |
| `Guid` | `uniqueidentifier` | 128-bit globally unique identifier |
| `byte[]` | `varbinary(max)` | Binary data, unlimited length |
| `enum` | `int` | Stored as the underlying integer value by default |

```ad-note
title: Enum Mapping Detail
By default, enums are stored as their **integer values** (`int` column). If you want to store them as **strings** instead (e.g., "Active" instead of 0), you must configure a value converter:

~~~csharp
// Store enum as string in the database
entity.Property(e => e.Status)
    .HasConversion<string>();
~~~

Or use the `[Column(TypeName = "nvarchar(50)")]` approach, combined with a converter.
```

### Precision and Scale for Numeric Types

| .NET Type | Default Precision | How to Override |
|---|---|---|
| `decimal` | `decimal(18,2)` | `[Column(TypeName = "decimal(10,4)")]` or `.HasPrecision(10, 4)` |
| `double` | `float` (53-bit) | Rarely needs overriding |
| `float` | `real` (24-bit) | Rarely needs overriding |

```ad-warning
title: decimal(18,2) May Truncate
The default `decimal(18,2)` means only **2 decimal places**. If you're storing currency that needs more precision, financial calculations, or scientific values, you **must** override this. For currency with 4 decimal places: `.HasPrecision(18, 4)` or `[Column(TypeName = "decimal(18,4)")]`.
```

---

## Column Nullability Convention

EF Core determines whether a column allows `NULL` based on the **nullability of the C# type**. This behavior changes depending on whether **Nullable Reference Types (NRT)** are enabled.

### With Nullable Reference Types Enabled (Recommended)

When NRT is enabled in your `.csproj`, EF Core respects the nullability annotations on reference types:

```xml
<!-- In your .csproj file -->
<PropertyGroup>
    <Nullable>enable</Nullable>
</PropertyGroup>
```

| C# Type | Column Nullability | Why |
|---|---|---|
| `string` | `NOT NULL` | Non-nullable reference type |
| `string?` | `NULL` allowed | Nullable reference type |
| `int` | `NOT NULL` | Value type (can't be null) |
| `int?` | `NULL` allowed | Nullable value type |
| `DateTime` | `NOT NULL` | Value type |
| `DateTime?` | `NULL` allowed | Nullable value type |
| `Customer` (navigation) | -- | Navigation nullability affects relationship optionality |

```csharp
// With NRT enabled (<Nullable>enable</Nullable>)
public class Customer
{
    public int Id { get; set; }               // NOT NULL (value type)
    public string Name { get; set; }          // NOT NULL (NRT: string is non-nullable)
    public string? MiddleName { get; set; }   // NULL allowed (NRT: string? is nullable)
    public int LoyaltyPoints { get; set; }    // NOT NULL (value type)
    public int? ReferralCode { get; set; }    // NULL allowed (nullable value type)
}
```

### Without Nullable Reference Types (Legacy Behavior)

When NRT is **not** enabled (the default before .NET 6 templates), all reference types are treated as **nullable** because the compiler doesn't distinguish `string` from `string?`:

| C# Type | Column Nullability |
|---|---|
| `string` | `NULL` allowed (reference type defaults to nullable) |
| `int` | `NOT NULL` |
| `int?` | `NULL` allowed |

```ad-tip
title: Always Enable NRT for New Projects
Starting with .NET 6, new project templates have NRT enabled by default. If you're working on an older project, strongly consider enabling it -- it gives you explicit control over column nullability and catches null reference bugs at compile time. Add `<Nullable>enable</Nullable>` to your `.csproj`.
```

### Nullability and Relationships

Column nullability directly affects whether a relationship is **required** or **optional**:

- `public int CustomerId { get; set; }` -- non-nullable FK means the relationship is **required** (every order MUST have a customer). Cascade delete is the default.
- `public int? CustomerId { get; set; }` -- nullable FK means the relationship is **optional** (an order CAN exist without a customer). ClientSetNull is the default.

See [[Key and Relationship Conventions]] for full details on how nullability affects cascade behavior.

---

## Complete Example: From POCO to Database

Here's a full example showing a POCO class and the exact table/columns EF Core generates by convention:

### The C# Code

```csharp
public class AppDbContext : DbContext
{
    public DbSet<BlogPost> BlogPosts { get; set; }  // DbSet property name = table name
}

public class BlogPost
{
    public int Id { get; set; }                  // PK, auto-increment
    public string Title { get; set; }            // nvarchar(max), NOT NULL
    public string? Subtitle { get; set; }        // nvarchar(max), nullable
    public string Content { get; set; }          // nvarchar(max), NOT NULL
    public DateTime PublishedDate { get; set; }  // datetime2(7), NOT NULL
    public bool IsPublished { get; set; }        // bit, NOT NULL
    public decimal Rating { get; set; }          // decimal(18,2), NOT NULL
    public int ViewCount { get; set; }           // int, NOT NULL
    public byte[] CoverImage { get; set; }       // varbinary(max), NOT NULL
    public Guid UniqueSlug { get; set; }         // uniqueidentifier, NOT NULL
    
    // Navigation property -- BlogPost belongs to an Author
    public int AuthorId { get; set; }            // FK, int, NOT NULL
    public Author Author { get; set; }           // Reference navigation

    // Excluded from database
    [NotMapped]
    public string Summary => Title + "...";      // Not a column
}
```

### The Generated SQL (SQL Server)

```sql
CREATE TABLE [BlogPosts] (
    [Id]            int              IDENTITY(1,1) NOT NULL PRIMARY KEY,
    [Title]         nvarchar(max)    NOT NULL,
    [Subtitle]      nvarchar(max)    NULL,
    [Content]       nvarchar(max)    NOT NULL,
    [PublishedDate]  datetime2(7)    NOT NULL,
    [IsPublished]   bit             NOT NULL,
    [Rating]        decimal(18,2)   NOT NULL,
    [ViewCount]     int             NOT NULL,
    [CoverImage]    varbinary(max)  NOT NULL,
    [UniqueSlug]    uniqueidentifier NOT NULL,
    [AuthorId]      int             NOT NULL,
    CONSTRAINT [FK_BlogPosts_Authors_AuthorId]
        FOREIGN KEY ([AuthorId]) REFERENCES [Authors]([Id])
        ON DELETE CASCADE
);

CREATE INDEX [IX_BlogPosts_AuthorId] ON [BlogPosts] ([AuthorId]);
```

```ad-note
title: Notice the Auto-Generated Index
EF Core automatically creates an **index on the FK column** (`AuthorId`). This is a convention that improves JOIN performance. You don't need to configure this manually.
```

---

## See Also

- [[Conventions Overview]] -- the big picture of conventions and when they're enough
- [[Key and Relationship Conventions]] -- how PKs, FKs, and relationships are discovered
- [[Overriding Conventions]] -- how to change table names, column names, and data types
- [[Entity Classes]] -- fundamentals of entity classes in EF Core
- [[Fluent API Configuration]] -- the most powerful way to override column and table conventions
