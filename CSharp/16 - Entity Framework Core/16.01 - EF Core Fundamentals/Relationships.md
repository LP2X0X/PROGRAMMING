---
tags: [csharp, ef-core, relationships, fundamentals]
---

## Overview

- **Relationships** in EF Core mirror [[Foreign Keys and Relationships|foreign key relationships]] in a relational database.
- EF Core uses **navigation properties** (C# object references) and **foreign key properties** to model relationships.
- Three relationship types: **One-to-Many**, **One-to-One**, and **Many-to-Many**.

### Key Terms

| Term                     | Meaning                                                        |
| ------------------------ | -------------------------------------------------------------- |
| **Principal entity**     | The "parent" — the entity that contains the PK being referenced |
| **Dependent entity**     | The "child" — the entity that contains the FK                   |
| **Navigation property**  | A C# property that references a related entity (or collection)  |
| **Foreign key property** | A scalar property (e.g., `int CustomerId`) that holds the FK value |

---

## One-to-Many (Most Common)

- One parent has **many** children. One child belongs to **one** parent.
- Example: one `Customer` has many `Order`s.

### Entity Classes

```csharp
// Principal (parent)
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }

    // Collection navigation — "one customer has many orders"
    public ICollection<Order> Orders { get; set; } = new List<Order>();
}

// Dependent (child)
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public decimal Total { get; set; }

    // Foreign key property
    public int CustomerId { get; set; }

    // Reference navigation — "this order belongs to one customer"
    public Customer Customer { get; set; }
}
```

### What EF Core Generates (SQL Server)

```sql
CREATE TABLE Customers (
    Id int IDENTITY(1,1) PRIMARY KEY,
    Name nvarchar(max) NOT NULL
);

CREATE TABLE Orders (
    Id int IDENTITY(1,1) PRIMARY KEY,
    OrderDate datetime2 NOT NULL,
    Total decimal(18,2) NOT NULL,
    CustomerId int NOT NULL,                             -- FK column
    FOREIGN KEY (CustomerId) REFERENCES Customers(Id)    -- FK constraint
        ON DELETE CASCADE
);
```

### Convention Breakdown

- `Customer` has `ICollection<Order> Orders` — EF Core sees "one customer, many orders."
- `Order` has `Customer Customer` — EF Core sees "this order points to one customer."
- `Order.CustomerId` matches the pattern `{NavigationName}Id` — EF Core uses it as the FK.
- If you omit the `CustomerId` property, EF Core creates a **shadow property** (FK exists in the DB but not in your C# class). It's better to include it explicitly so you can filter by `CustomerId` without loading the full `Customer` object.

```ad-tip
**Always include the FK property explicitly** (e.g., `public int CustomerId { get; set; }`). This lets you do things like `db.Orders.Where(o => o.CustomerId == 5)` without needing a JOIN to the Customers table.
```

### Fluent API Configuration

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Order>()
        .HasOne(o => o.Customer)          // Order has one Customer
        .WithMany(c => c.Orders)          // Customer has many Orders
        .HasForeignKey(o => o.CustomerId) // FK is Order.CustomerId
        .OnDelete(DeleteBehavior.Cascade); // delete customer → delete their orders
}
```

---

## One-to-One

- Each entity on both sides references **exactly one** of the other.
- Example: each `Employee` has one `EmployeeProfile`, and each profile belongs to one employee.
- The **dependent** side (the one with the FK) must be determined — EF Core sometimes can't figure this out by convention alone.

### Entity Classes

```csharp
// Principal
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }

    // Reference navigation
    public EmployeeProfile Profile { get; set; }
}

// Dependent (has the FK)
public class EmployeeProfile
{
    public int Id { get; set; }
    public string Bio { get; set; }
    public string PhotoUrl { get; set; }

    // Foreign key — points to Employee
    public int EmployeeId { get; set; }

    // Reference navigation
    public Employee Employee { get; set; }
}
```

### Fluent API Configuration

```csharp
modelBuilder.Entity<Employee>()
    .HasOne(e => e.Profile)                // Employee has one Profile
    .WithOne(p => p.Employee)              // Profile has one Employee
    .HasForeignKey<EmployeeProfile>(p => p.EmployeeId);  // FK is on EmployeeProfile
```

```ad-note
title: You must specify which side has the FK
In one-to-one relationships, EF Core requires you to specify `HasForeignKey<DependentType>(...)` because either side *could* hold the FK. If you don't, you'll get a runtime error or an unexpected shadow FK.
```

---

## Many-to-Many

- Both sides can reference **many** of the other.
- Example: `Student` can enroll in many `Course`s, and each `Course` has many `Student`s.
- In the database, this requires a **join table** (junction table) to hold the FK pairs.

### EF Core 5+ — Automatic Join Table (Skip Navigation)

Starting with EF Core 5, you can define a many-to-many relationship **without creating a join entity class**. EF Core creates the join table for you.

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }

    // Collection navigation — many courses
    public ICollection<Course> Courses { get; set; } = new List<Course>();
}

public class Course
{
    public int Id { get; set; }
    public string Title { get; set; }

    // Collection navigation — many students
    public ICollection<Student> Students { get; set; } = new List<Student>();
}
```

- EF Core automatically creates a join table named `CourseStudent` with columns `CoursesId` and `StudentsId`.

### Explicit Join Entity (When You Need Extra Data on the Relationship)

If the join table needs **its own columns** (e.g., enrollment date, grade), you must create an explicit join entity.

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<Enrollment> Enrollments { get; set; } = new List<Enrollment>();
}

public class Course
{
    public int Id { get; set; }
    public string Title { get; set; }
    public ICollection<Enrollment> Enrollments { get; set; } = new List<Enrollment>();
}

// Explicit join entity with extra data
public class Enrollment
{
    public int StudentId { get; set; }      // FK + composite PK part
    public Student Student { get; set; }

    public int CourseId { get; set; }        // FK + composite PK part
    public Course Course { get; set; }

    public DateTime EnrolledDate { get; set; }  // extra data on the relationship
    public string? Grade { get; set; }           // extra data on the relationship
}
```

```csharp
// Fluent API for explicit join entity
modelBuilder.Entity<Enrollment>()
    .HasKey(e => new { e.StudentId, e.CourseId });  // composite PK

modelBuilder.Entity<Enrollment>()
    .HasOne(e => e.Student)
    .WithMany(s => s.Enrollments)
    .HasForeignKey(e => e.StudentId);

modelBuilder.Entity<Enrollment>()
    .HasOne(e => e.Course)
    .WithMany(c => c.Enrollments)
    .HasForeignKey(e => e.CourseId);
```

```ad-warning
The automatic join table (skip navigation) is clean but limited — you **cannot add extra columns** to it. The moment you need any data on the relationship itself (date, status, sort order), switch to an explicit join entity. This is a very common requirement in real-world apps.
```

---

## Foreign Key Conventions Summary

| Pattern                                         | EF Core Interpretation               |
| ------------------------------------------------ | ------------------------------------ |
| `public int CustomerId { get; set; }`             | FK to `Customer` (matches `{Type}Id`) |
| `public int CustomerFk { get; set; }`             | **Not** recognized — need `[ForeignKey]` or Fluent API |
| `public Customer Customer { get; set; }`          | Reference navigation to `Customer`   |
| `public ICollection<Order> Orders { get; set; }`  | Collection navigation (one-to-many or many-to-many) |

---

## Fluent API Cheat Sheet

| Method                          | Purpose                                             |
| ------------------------------- | --------------------------------------------------- |
| `.HasOne(e => e.Nav)`           | This entity has one related entity                  |
| `.HasMany(e => e.Nav)`          | This entity has many related entities               |
| `.WithOne(e => e.Nav)`          | The other side has one (completing one-to-one or many-to-one) |
| `.WithMany(e => e.Nav)`         | The other side has many (completing one-to-many)    |
| `.HasForeignKey(e => e.FK)`     | Specify which property is the FK                    |
| `.HasForeignKey<TDependent>()` | Specify FK with explicit dependent type (one-to-one) |
| `.IsRequired()`                 | FK is NOT NULL (required relationship)              |
| `.OnDelete(DeleteBehavior.X)`   | Cascade delete behavior                             |

### Reading the Fluent API

Read it as an English sentence:

```csharp
modelBuilder.Entity<Order>()
    .HasOne(o => o.Customer)       // "An Order has one Customer"
    .WithMany(c => c.Orders)      // "and that Customer has many Orders"
    .HasForeignKey(o => o.CustomerId);  // "linked by Order.CustomerId"
```

---

## Cascade Delete Behavior

- **Cascade delete** controls what happens to dependent (child) rows when the principal (parent) row is deleted.

| DeleteBehavior    | Required relationship (non-nullable FK) | Optional relationship (nullable FK) |
| ----------------- | --------------------------------------- | ----------------------------------- |
| `Cascade`         | Delete children when parent is deleted  | Delete children when parent is deleted |
| `Restrict`        | **Throw exception** — must delete children first | **Throw exception**              |
| `SetNull`         | N/A (FK is non-nullable)                | Set FK to NULL                      |
| `NoAction`        | Database throws FK violation error      | Database throws FK violation error  |
| `ClientSetNull`   | N/A                                     | Set FK to NULL for tracked entities only |

- **Default behavior:**
  - Required relationships (non-nullable FK) → `Cascade`
  - Optional relationships (nullable FK) → `ClientSetNull`

```csharp
modelBuilder.Entity<Order>()
    .HasOne(o => o.Customer)
    .WithMany(c => c.Orders)
    .HasForeignKey(o => o.CustomerId)
    .OnDelete(DeleteBehavior.Restrict);  // prevent accidental cascade deletes
```

```ad-warning
`Cascade` is the default for required relationships, which means deleting a Customer will silently delete **all** their Orders. In many real-world applications, you want `Restrict` instead — it forces you to handle children explicitly and prevents accidental data loss.

See also [[Foreign Key|SQL-level ON DELETE behavior]].
```

---

## See Also

- [[Entity Classes]] — configuring the classes that participate in relationships
- [[DbContext]] — where relationship configuration lives (`OnModelCreating`)
- [[EF Core Overview]] — the big picture
- [[Foreign Keys and Relationships]] — the SQL-level view (from Database vault)
- [[Foreign Key]] — SQL constraint details and referential actions
