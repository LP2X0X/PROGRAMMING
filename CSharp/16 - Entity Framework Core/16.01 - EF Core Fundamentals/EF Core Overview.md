---
tags: [csharp, ef-core, orm, fundamentals]
---

## What Is EF Core

- **Entity Framework Core** (EF Core) is Microsoft's official **ORM** (Object-Relational Mapper) for .NET. It lets you work with a database using C# objects instead of writing raw SQL.
- An ORM maps **classes to tables**, **properties to columns**, and **object instances to rows**. You write LINQ queries against your C# objects, and EF Core translates them into SQL automatically.

```ad-note
title: ORM = Object-Relational Mapper
An ORM bridges the mismatch between your object-oriented C# code and the relational (table-based) structure of a database. Without an ORM, you'd manually convert between `DbDataReader` rows and your classes — EF Core does this for you.
```

---

## How EF Core Sits on Top of ADO.NET

- EF Core is **not a replacement** for [[ADO.NET Overview|ADO.NET]] — it is **built on top of it**.
- Under the hood, EF Core uses the same ADO.NET primitives you'd use manually:

| ADO.NET Component       | What EF Core Does With It                                |
| ----------------------- | -------------------------------------------------------- |
| `DbConnection`          | Opens/closes the connection to the database              |
| `DbCommand`             | Sends the generated SQL to the database                  |
| `DbDataReader`          | Reads rows returned by queries                           |
| `DbTransaction`         | Wraps `SaveChanges()` in an implicit transaction         |
| `DbParameter`           | Parameterizes values to prevent SQL injection            |

- The key difference: with ADO.NET you write all of this yourself. With EF Core, you write LINQ and it generates the ADO.NET calls for you.

### Architecture Flow

```
Your C# Code
    │  (you write LINQ queries against DbSet<T>)
    ▼
EF Core (LINQ Provider)
    │  (translates LINQ expression tree → SQL)
    ▼
ADO.NET (DbConnection / DbCommand)
    │  (sends SQL over the wire, reads results)
    ▼
Database (SQL Server, PostgreSQL, SQLite, etc.)
```

---

## EF Core vs ADO.NET — When to Use Which

| Aspect               | ADO.NET (raw)                                  | EF Core                                           |
| --------------------- | ---------------------------------------------- | ------------------------------------------------- |
| **SQL**               | You write SQL manually                         | EF Core generates SQL from LINQ                   |
| **Object mapping**    | You map `DbDataReader` fields to objects by hand | Automatic — columns map to properties              |
| **Performance**       | Fastest — no abstraction overhead              | Slower — LINQ translation + change tracking cost   |
| **Control**           | Full control over every SQL statement          | Convention-based, less granular control             |
| **Productivity**      | More code, more boilerplate                    | Less code, rapid development                       |
| **Change tracking**   | You track changes yourself                     | Built-in — detects what changed and generates UPDATE/INSERT/DELETE |
| **Migrations**        | You manage schema changes manually             | `dotnet ef migrations add` generates migration scripts |
| **Learning curve**    | SQL knowledge required                         | LINQ knowledge required, SQL knowledge still helps |

```ad-tip
**Use EF Core** for typical CRUD business applications where developer productivity matters more than squeezing out every last millisecond.

**Use ADO.NET** (or Dapper) for performance-critical paths — bulk inserts, reporting queries, tight loops — where you need exact SQL and minimal overhead.

Many real-world projects use **both**: EF Core for general CRUD, raw ADO.NET or Dapper for hot paths.
```

---

## NuGet Packages You Need

Every EF Core project needs at least **two packages**: the core library and a **database provider**.

| Package                                          | Purpose                              |
| ------------------------------------------------ | ------------------------------------ |
| `Microsoft.EntityFrameworkCore`                  | Core EF library (always required)    |
| `Microsoft.EntityFrameworkCore.SqlServer`         | SQL Server / Azure SQL provider      |
| `Microsoft.EntityFrameworkCore.Sqlite`            | SQLite provider                      |
| `Npgsql.EntityFrameworkCore.PostgreSQL`           | PostgreSQL provider                  |
| `Pomelo.EntityFrameworkCore.MySql`                | MySQL / MariaDB provider             |
| `Microsoft.EntityFrameworkCore.Tools`             | `dotnet ef` CLI (migrations, scaffolding) |
| `Microsoft.EntityFrameworkCore.Design`            | Design-time services (required for migrations) |

```
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
```

```ad-warning
`Microsoft.EntityFrameworkCore.Tools` and `Microsoft.EntityFrameworkCore.Design` are **development-time only** packages. They are not needed at runtime and can be marked with `<PrivateAssets>all</PrivateAssets>` in the `.csproj` to prevent them from being published with your app.
```

---

## Minimal Working Example

```csharp
// 1. Define your entity (maps to a table)
public class Product
{
    public int Id { get; set; }          // PK by convention
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// 2. Define your DbContext (represents the database session)
public class AppDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }  // maps to "Products" table

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlServer("Server=.;Database=MyApp;Trusted_Connection=true;");
    }
}

// 3. Use it
using var db = new AppDbContext();

// INSERT
db.Products.Add(new Product { Name = "Widget", Price = 9.99m });
db.SaveChanges();  // generates INSERT INTO Products ...

// SELECT
var cheap = db.Products
    .Where(p => p.Price < 20)   // translated to WHERE Price < 20
    .ToList();
```

- `DbSet<Product>` exposes `IQueryable<Product>` — LINQ methods like `.Where()`, `.OrderBy()`, `.Select()` are translated to SQL.
- `SaveChanges()` wraps all pending changes in a **single transaction** and sends the generated INSERT/UPDATE/DELETE to the database via ADO.NET.

---

## See Also

- [[DbContext]] — the central class explained in detail
- [[Entity Classes]] — how to define and configure entity classes
- [[Database Providers]] — how EF Core connects to different databases
- [[Relationships]] — configuring one-to-many, one-to-one, many-to-many
- [[ADO.NET Overview]] — the lower-level data access EF Core is built on
- [[DbDataReader]] — the raw row reader EF Core uses internally
