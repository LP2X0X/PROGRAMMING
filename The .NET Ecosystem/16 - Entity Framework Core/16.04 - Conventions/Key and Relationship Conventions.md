---
tags: [csharp, ef-core, conventions, keys, relationships]
---

## Primary Key Convention

EF Core automatically identifies a property as the **primary key** if its name matches one of two patterns:

1. **`Id`** -- a property simply named `Id` on any class
2. **`{ClassName}Id`** -- the class name followed by `Id` (e.g., `CustomerId` on a `Customer` class)

The matching is **case-insensitive** -- `ID`, `id`, `iD`, `Id` all work. Both `Id` and `CustomerId` are valid for a class named `Customer`.

```csharp
public class Customer
{
    public int Id { get; set; }            // ✅ PK by convention -- matches "Id"
}

public class Order
{
    public int OrderId { get; set; }       // ✅ PK by convention -- matches "{ClassName}Id"
}

public class Product
{
    public int ProductCode { get; set; }   // ❌ NOT a PK -- doesn't match Id or ProductId
    // EF Core will throw at runtime: "The entity type 'Product' requires a primary key"
}
```

```ad-warning
title: Both Patterns on One Class
If a class has BOTH an `Id` property AND a `{ClassName}Id` property, the `Id` property wins. But this is confusing and should be avoided.

~~~csharp
public class Customer
{
    public int Id { get; set; }              // ← This becomes the PK
    public int CustomerId { get; set; }      // ← This is treated as a regular column
}
~~~
```

---

## Value Generation (Auto-Increment / Auto-Generated PKs)

When EF Core discovers a primary key, it also decides whether the database should **auto-generate** the value. This depends on the PK's .NET type:

| PK Type | Value Generation Strategy | SQL Server Implementation | Must You Set the Value? |
|---|---|---|---|
| `int` | **Identity** (auto-increment) | `IDENTITY(1,1)` | No -- database generates it on INSERT |
| `long` | **Identity** (auto-increment) | `IDENTITY(1,1)` | No -- database generates it |
| `short` | **Identity** (auto-increment) | `IDENTITY(1,1)` | No -- database generates it |
| `Guid` | **Database-generated** | `newsequentialid()` as default | No -- database generates it |
| `string` | **None** | No auto-generation | **Yes** -- you must set it before saving |
| `byte[]` | **None** | No auto-generation | **Yes** -- you must set it |

```csharp
public class Order
{
    public int Id { get; set; }   // Auto-increment -- EF Core handles it
}

var order = new Order { /* don't set Id */ };
context.Orders.Add(order);
await context.SaveChangesAsync();
// order.Id is now 1 (or whatever the database assigned)
```

```ad-warning
title: String PKs Require Manual Value Assignment
If your PK is a `string`, EF Core will NOT auto-generate it. You **must** set the value yourself before calling `SaveChanges()`. Forgetting this causes a runtime exception.

~~~csharp
public class Country
{
    public string Id { get; set; }      // String PK -- no auto-generation
    public string Name { get; set; }
}

// You MUST set the PK before saving
var country = new Country { Id = "US", Name = "United States" };
context.Countries.Add(country);
await context.SaveChangesAsync();
~~~
```

### Client-Side Guid Generation

While SQL Server generates `Guid` PKs server-side with `newsequentialid()`, you can also generate them **client-side** before saving. EF Core will use your value instead of asking the database to generate one:

```csharp
var entity = new MyEntity { Id = Guid.NewGuid() };
context.Add(entity);
await context.SaveChangesAsync();
// EF Core sends your Guid to the database instead of relying on newsequentialid()
```

```ad-tip
title: Sequential Guids for Performance
`Guid.NewGuid()` generates random GUIDs that cause **index fragmentation** in SQL Server (clustered index on PK). `newsequentialid()` generates sequential ones that insert in order. If you generate client-side, consider libraries like `RT.Comb` or `UlidNet` that produce sequential identifiers to avoid fragmentation.
```

---

## Composite Keys

```ad-warning
title: Conventions CANNOT Discover Composite Keys
This is one of the most important limitations. If your table has a composite primary key (PK made of multiple columns), conventions will **not** detect it. You **must** use the Fluent API.
```

```csharp
// This entity has a composite PK -- conventions can't detect this
public class OrderItem
{
    public int OrderId { get; set; }
    public int ProductId { get; set; }
    public int Quantity { get; set; }

    public Order Order { get; set; }
    public Product Product { get; set; }
}

// You MUST configure it in OnModelCreating
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<OrderItem>()
        .HasKey(oi => new { oi.OrderId, oi.ProductId });  // Composite PK
}
```

Data Annotations also **cannot** define composite keys. The `[Key]` attribute only marks a single property. For composite keys, Fluent API is the only option.

---

## Foreign Key Convention

EF Core automatically identifies a property as a **foreign key** if its name matches either of these patterns:

1. **`{NavigationPropertyName}Id`** -- the name of the navigation property followed by `Id`
2. **`{PrincipalTypeName}Id`** -- the name of the related (principal) entity type followed by `Id`

```csharp
public class Order
{
    public int Id { get; set; }

    // Pattern 1: {NavigationPropertyName}Id
    public int CustomerId { get; set; }        // FK -- matches "Customer" + "Id"
    public Customer Customer { get; set; }     // Navigation property named "Customer"

    // Pattern 2: {PrincipalTypeName}Id (also works without a matching nav name)
    public int ShippingAddressId { get; set; } // FK -- matches "Address" + "Id"? No!
    // ⚠️ This only matches if there's an "Address" navigation or "ShippingAddress" navigation
}
```

### FK Naming Examples

| Navigation Property | FK Property | Matches? | Pattern |
|---|---|---|---|
| `Customer Customer` | `CustomerId` | Yes | Both Pattern 1 and 2 (same name) |
| `Customer Buyer` | `BuyerId` | Yes | Pattern 1: `{Buyer}Id` |
| `Customer Buyer` | `CustomerId` | Yes | Pattern 2: `{Customer}Id` |
| `Customer Buyer` | `ClientId` | No | Doesn't match navigation or type name |

```ad-note
title: FK Property Type Must Match PK Type
The foreign key property's type must match the principal entity's primary key type. If `Customer.Id` is `int`, then `Order.CustomerId` must also be `int` (or `int?` for an optional relationship). A type mismatch causes a runtime error during model building.
```

---

## Relationship Discovery

EF Core discovers relationships by examining **navigation properties** -- properties whose type is another entity class (reference navigation) or a collection of entity classes (collection navigation).

### Reference Navigation (Points to One Entity)

A property whose type is another entity class. Indicates the "many" side of a one-to-many or one side of a one-to-one.

```csharp
public class Order
{
    public int Id { get; set; }
    public Customer Customer { get; set; }     // Reference navigation → "this order has one customer"
}
```

### Collection Navigation (Points to Many Entities)

A property that is an `ICollection<T>`, `IList<T>`, `List<T>`, `IEnumerable<T>`, or `HashSet<T>` of another entity. Indicates the "one" side of a one-to-many.

```csharp
public class Customer
{
    public int Id { get; set; }
    public ICollection<Order> Orders { get; set; }  // Collection navigation → "one customer has many orders"
        = new List<Order>();
}
```

### Pairing Navigations

When **both sides** of a relationship have navigation properties, EF Core **automatically pairs them**:

```csharp
// EF Core sees these two navigations and pairs them into one relationship:
// Customer.Orders (collection) ←→ Order.Customer (reference)

public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Order> Orders { get; set; } = new List<Order>();  // "One" side
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }         // FK
    public Customer Customer { get; set; }      // "Many" side -- paired with Customer.Orders
}
```

```ad-warning
title: Ambiguous Relationships
If an entity has **two or more navigation properties** pointing to the same type, EF Core **cannot** automatically pair them and will throw an error. You must resolve the ambiguity with `[InverseProperty]` or Fluent API.

~~~csharp
public class Person
{
    public int Id { get; set; }

    // Two navigations to the same type -- EF Core can't pair them automatically
    public ICollection<Message> SentMessages { get; set; }
    public ICollection<Message> ReceivedMessages { get; set; }
}

public class Message
{
    public int Id { get; set; }

    public int SenderId { get; set; }
    [InverseProperty("SentMessages")]      // ← Required to resolve ambiguity
    public Person Sender { get; set; }

    public int ReceiverId { get; set; }
    [InverseProperty("ReceivedMessages")]  // ← Required to resolve ambiguity
    public Person Receiver { get; set; }
}
~~~
```

### One-Sided Navigation (Only One Side Defined)

You don't need both sides. EF Core can work with just one navigation property:

```csharp
// Only the dependent has a navigation -- no collection on Customer
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public Customer Customer { get; set; }   // Only this side exists
}

public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    // No ICollection<Order> -- that's fine. The relationship still exists.
}
```

---

## Shadow Foreign Key Properties

If a navigation property exists but **no FK property** is declared in C#, EF Core creates a **shadow property** -- a property that exists in the EF model and in the database but **not in your C# class**.

```csharp
public class Order
{
    public int Id { get; set; }
    public Customer Customer { get; set; }   // Navigation exists
    // No CustomerId property declared!
}

// EF Core automatically creates a shadow property named "CustomerId" (int)
// It exists in the database as a column, but you can't access it directly in C#
```

### Accessing Shadow Properties

You can query shadow properties through `EF.Property<T>()`:

```csharp
// Query using the shadow FK property
var orders = context.Orders
    .Where(o => EF.Property<int>(o, "CustomerId") == 42)
    .ToList();
```

```ad-tip
title: Prefer Explicit FK Properties
While shadow properties work, it's **strongly recommended** to declare FK properties explicitly in your entity classes. This gives you:
- Direct access to the FK value without loading the related entity (e.g., `order.CustomerId` instead of `order.Customer.Id`)
- Simpler LINQ queries (no `EF.Property<>()` needed)
- Clearer intent in your code
```

---

## Cascade Delete Defaults

EF Core sets cascade behavior based on whether the relationship is **required** (non-nullable FK) or **optional** (nullable FK):

| Relationship Type | FK Nullability | Default Delete Behavior | What Happens When Principal Is Deleted |
|---|---|---|---|
| **Required** | `int CustomerId` (non-nullable) | `Cascade` | Dependent rows are **deleted automatically** |
| **Optional** | `int? CustomerId` (nullable) | `ClientSetNull` | FK is set to `null` on tracked dependents. Untracked dependents cause a DB constraint violation. |

### Required Relationship (Cascade Delete)

```csharp
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }        // Non-nullable FK → REQUIRED relationship
    public Customer Customer { get; set; }
}

// When you delete a Customer, all their Orders are deleted too
context.Customers.Remove(customer);
await context.SaveChangesAsync();
// DELETE FROM Orders WHERE CustomerId = @id
// DELETE FROM Customers WHERE Id = @id
```

### Optional Relationship (ClientSetNull)

```csharp
public class Order
{
    public int Id { get; set; }
    public int? CustomerId { get; set; }       // Nullable FK → OPTIONAL relationship
    public Customer? Customer { get; set; }
}

// When you delete a Customer, tracked Orders get their FK set to NULL
// BUT: if the Order is not tracked by the context, you get a database exception
```

```ad-warning
title: ClientSetNull vs SetNull
`ClientSetNull` only sets the FK to `null` on entities **currently tracked** by the DbContext. If there are dependent rows in the database that are NOT loaded into the context, the database will throw a foreign key constraint violation. If you want the database itself to handle the nulling (regardless of tracking), configure `DeleteBehavior.SetNull` via Fluent API:

~~~csharp
modelBuilder.Entity<Order>()
    .HasOne(o => o.Customer)
    .WithMany(c => c.Orders)
    .OnDelete(DeleteBehavior.SetNull);
~~~
```

### All Delete Behaviors

| Behavior | Tracked Dependents | Untracked Dependents in DB |
|---|---|---|
| `Cascade` | Deleted | Deleted (DB handles it) |
| `ClientSetNull` | FK set to null | **Exception** (DB constraint violation) |
| `SetNull` | FK set to null | FK set to null (DB handles it) |
| `Restrict` | **Exception** | **Exception** |
| `NoAction` | Nothing | Database decides (usually exception) |
| `ClientNoAction` | Nothing | Database decides |
| `ClientCascade` | Deleted | **Exception** |

---

## Relationship Type Summary

Here's how EF Core determines the relationship type from your navigation properties and FK setup:

### One-to-Many (Most Common)

```csharp
// One Customer has many Orders
public class Customer
{
    public int Id { get; set; }
    public ICollection<Order> Orders { get; set; }     // Collection nav
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }                 // FK
    public Customer Customer { get; set; }              // Reference nav
}
```

### One-to-One

```csharp
// One Customer has one Address (and vice versa)
public class Customer
{
    public int Id { get; set; }
    public Address Address { get; set; }                // Reference nav (no collection)
}

public class Address
{
    public int Id { get; set; }
    public int CustomerId { get; set; }                 // FK on the dependent side
    public Customer Customer { get; set; }              // Reference nav
}
```

```ad-note
title: One-to-One Requires Careful Setup
For one-to-one relationships, EF Core needs to know which entity is the **dependent** (the one with the FK). If both sides have reference navigations and no FK property, EF Core may pick the wrong side. Always define the FK property explicitly on the dependent.
```

### Many-to-Many (EF Core 5+)

```csharp
// Many Students can enroll in many Courses
public class Student
{
    public int Id { get; set; }
    public ICollection<Course> Courses { get; set; }    // Collection nav
}

public class Course
{
    public int Id { get; set; }
    public ICollection<Student> Students { get; set; }  // Collection nav
}

// EF Core 5+ automatically creates a join table (StudentCourse or CourseStudent)
// with StudentId and CourseId as a composite PK and two FKs
```

---

## Complete Convention Example

```csharp
// Everything discovered by convention -- zero configuration
public class SchoolDbContext : DbContext
{
    public DbSet<Student> Students { get; set; }
    public DbSet<Course> Courses { get; set; }
    public DbSet<Enrollment> Enrollments { get; set; }
}

public class Student
{
    public int Id { get; set; }                             // PK (auto-increment)
    public string Name { get; set; }                        // nvarchar(max), NOT NULL
    public DateTime EnrollmentDate { get; set; }            // datetime2, NOT NULL

    public ICollection<Enrollment> Enrollments { get; set; } // One-to-many
        = new List<Enrollment>();
}

public class Course
{
    public int CourseId { get; set; }                        // PK ({ClassName}Id pattern)
    public string Title { get; set; }                       // nvarchar(max), NOT NULL
    public int Credits { get; set; }                        // int, NOT NULL

    public ICollection<Enrollment> Enrollments { get; set; } // One-to-many
        = new List<Enrollment>();
}

public class Enrollment
{
    public int Id { get; set; }                              // PK
    public int StudentId { get; set; }                       // FK (required, cascade)
    public Student Student { get; set; }                     // Reference nav
    public int CourseId { get; set; }                        // FK (required, cascade)
    public Course Course { get; set; }                       // Reference nav
    public string? Grade { get; set; }                       // nvarchar(max), nullable
}
```

This generates three tables with proper PKs, FKs, indexes, and cascade delete behavior -- all from conventions alone.

---

## See Also

- [[Conventions Overview]] -- the big picture of all conventions
- [[Table and Column Conventions]] -- table naming, column types, and nullability rules
- [[Overriding Conventions]] -- how to override PK, FK, and relationship conventions
- [[Relationships]] -- in-depth coverage of all relationship types with more examples
- [[Fluent API Configuration]] -- the most powerful way to configure keys and relationships
