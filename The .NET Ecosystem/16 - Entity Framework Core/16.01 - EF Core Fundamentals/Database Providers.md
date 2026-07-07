---
tags: [csharp, ef-core, providers, fundamentals]
---

## What Is a Database Provider

- A **database provider** is a plugin that teaches EF Core how to talk to a specific database engine.
- EF Core itself is **database-agnostic** — it generates an abstract query plan. The provider translates that plan into database-specific SQL and handles the underlying ADO.NET [[Data Providers|data provider]] (`DbConnection`, `DbCommand`, etc.).
- You always need **exactly one provider** NuGet package in addition to the core `Microsoft.EntityFrameworkCore` package.

```
EF Core (database-agnostic)
    │
    ├── SqlServer Provider  → generates T-SQL  → uses SqlConnection (ADO.NET)
    ├── SQLite Provider     → generates SQLite SQL → uses SqliteConnection
    ├── Npgsql Provider     → generates PostgreSQL → uses NpgsqlConnection
    └── Pomelo Provider     → generates MySQL SQL  → uses MySqlConnection
```

---

## Common Providers

| Database       | NuGet Package                                   | `UseXxx()` Method       | Maintained By        |
| -------------- | ----------------------------------------------- | ----------------------- | -------------------- |
| SQL Server     | `Microsoft.EntityFrameworkCore.SqlServer`        | `UseSqlServer()`        | Microsoft            |
| SQLite         | `Microsoft.EntityFrameworkCore.Sqlite`           | `UseSqlite()`           | Microsoft            |
| PostgreSQL     | `Npgsql.EntityFrameworkCore.PostgreSQL`          | `UseNpgsql()`           | Npgsql team          |
| MySQL / MariaDB| `Pomelo.EntityFrameworkCore.MySql`               | `UseMySql()`            | Pomelo Foundation    |
| In-Memory      | `Microsoft.EntityFrameworkCore.InMemory`         | `UseInMemoryDatabase()` | Microsoft            |
| Cosmos DB      | `Microsoft.EntityFrameworkCore.Cosmos`           | `UseCosmos()`           | Microsoft            |
| Oracle         | `Oracle.EntityFrameworkCore`                     | `UseOracle()`           | Oracle               |

```ad-note
title: MySQL — Pomelo vs Oracle's Provider
There are two MySQL providers: **Pomelo** (`Pomelo.EntityFrameworkCore.MySql`) and Oracle's official one (`MySql.EntityFrameworkCore`). **Pomelo is the community standard** — it has better EF Core feature support, more active development, and wider adoption. Use Pomelo unless you have a specific reason for Oracle's provider.
```

---

## How to Switch Providers

Switching databases is primarily a two-step process:

1. **Swap the NuGet package** (remove the old one, install the new one)
2. **Change the `UseXxx()` call** and update the connection string

### SQL Server

```
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder options)
{
    options.UseSqlServer(
        "Server=.;Database=MyApp;Trusted_Connection=true;TrustServerCertificate=true;");
}
```

### SQLite

```
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
```

```csharp
options.UseSqlite("Data Source=myapp.db");
```

### PostgreSQL

```
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```

```csharp
options.UseNpgsql("Host=localhost;Database=myapp;Username=postgres;Password=secret");
```

### MySQL / MariaDB

```
dotnet add package Pomelo.EntityFrameworkCore.MySql
```

```csharp
options.UseMySql(
    "Server=localhost;Database=myapp;User=root;Password=secret;",
    new MySqlServerVersion(new Version(8, 0, 36))  // specify your MySQL version
);
// For MariaDB:
// new MariaDbServerVersion(new Version(11, 4, 0))
```

```ad-warning
**Switching providers doesn't mean your app is automatically portable.** Some things that differ between databases:
- **Auto-increment syntax** — `IDENTITY` (SQL Server) vs `AUTO_INCREMENT` (MySQL) vs `SERIAL` (PostgreSQL)
- **String types** — `nvarchar(max)` vs `TEXT` vs `varchar`
- **Date types** — `datetime2` vs `TIMESTAMP` vs `timestamptz`
- **Raw SQL queries** — if you use `.FromSqlRaw()` or `.ExecuteSqlRaw()`, that SQL is database-specific

EF Core handles most of these differences through the provider, but raw SQL and some advanced features may break when switching.
```

---

## Connection String Differences by Provider

| Provider    | Connection String Example                                                         |
| ----------- | --------------------------------------------------------------------------------- |
| SQL Server  | `Server=.;Database=MyApp;Trusted_Connection=true;TrustServerCertificate=true;`   |
| SQL Server (remote) | `Server=myserver.db.windows.net;Database=MyApp;User Id=sa;Password=P@ss;` |
| SQLite      | `Data Source=myapp.db` (file-based)                                               |
| SQLite (in-memory) | `Data Source=:memory:` (lives only while connection is open)                |
| PostgreSQL  | `Host=localhost;Port=5432;Database=myapp;Username=postgres;Password=secret;`      |
| MySQL       | `Server=localhost;Port=3306;Database=myapp;User=root;Password=secret;`            |

- For more, see [[Connection Strings]].

---

## In-Memory Provider (for Testing)

- The **In-Memory provider** stores data in memory (no actual database). Useful for **unit tests** where you want to test your EF Core logic without a real database.

```
dotnet add package Microsoft.EntityFrameworkCore.InMemory
```

```csharp
// In a test
var options = new DbContextOptionsBuilder<AppDbContext>()
    .UseInMemoryDatabase(databaseName: "TestDb")  // each name = isolated DB
    .Options;

using var db = new AppDbContext(options);

db.Products.Add(new Product { Name = "Test Widget", Price = 5.00m });
db.SaveChanges();

var product = db.Products.First();
Assert.Equal("Test Widget", product.Name);
```

```ad-warning
**The In-Memory provider has significant limitations:**
- It does **not** enforce foreign key constraints or referential integrity
- It does **not** support transactions
- It does **not** support raw SQL queries
- LINQ queries run in-memory (C#), not translated to SQL — so a query that works in-memory might fail against a real database

For **integration tests** that need realistic behavior, use **SQLite in-memory mode** (`Data Source=:memory:`) instead — it's a real database engine with proper constraint enforcement.
```

### SQLite In-Memory (Better Testing Alternative)

```csharp
// SQLite in-memory — more realistic than InMemory provider
var connection = new SqliteConnection("Data Source=:memory:");
connection.Open();  // keep connection open — closing it destroys the DB

var options = new DbContextOptionsBuilder<AppDbContext>()
    .UseSqlite(connection)
    .Options;

using var db = new AppDbContext(options);
db.Database.EnsureCreated();  // create tables from your model

// Now you have a real database engine with proper constraints
db.Products.Add(new Product { Name = "Test Widget", Price = 5.00m });
db.SaveChanges();
```

---

## Choosing a Provider — Quick Decision Guide

| Scenario                                    | Recommended Provider           |
| ------------------------------------------- | ------------------------------ |
| Enterprise / .NET shop / Azure              | SQL Server                     |
| Open-source project / Linux-first           | PostgreSQL                     |
| Embedded / desktop / mobile app             | SQLite                         |
| Existing MySQL/MariaDB infrastructure       | MySQL (Pomelo)                 |
| Unit tests (fast, no external dependencies) | SQLite in-memory or InMemory   |
| Learning / prototyping                      | SQLite (simplest setup)        |

---

## See Also

- [[EF Core Overview]] — how providers fit in the EF Core architecture
- [[DbContext]] — where you configure the provider (`OnConfiguring` or DI)
- [[Connection Strings]] — full connection string reference for each database
- [[Data Providers]] — the ADO.NET-level providers that EF Core providers wrap
