---
tags: [csharp, ef-core, dbcontext, fundamentals]
---

## What Is DbContext

- **`DbContext`** is the central class in EF Core. It represents a **session with the database** and acts as a **Unit of Work**.
- It is responsible for:
  - **Querying** — translating LINQ to SQL and materializing results into objects
  - **Change tracking** — detecting which entities have been added, modified, or deleted
  - **Saving** — generating and executing INSERT/UPDATE/DELETE SQL via `SaveChanges()`
  - **Caching** — holding a first-level identity cache so the same row isn't loaded twice in one session

- Under the hood, `DbContext` **wraps a `DbConnection`** (the same ADO.NET connection you'd use manually). It opens the connection when needed and closes it when disposed.

```ad-note
title: DbContext = Unit of Work + Repository
A single `DbContext` instance represents one logical unit of work. You load entities, modify them, then call `SaveChanges()` once. All changes are sent to the database in a **single transaction**.
```

---

## Creating a Basic DbContext

```csharp
public class AppDbContext : DbContext
{
    // Each DbSet<T> maps to a database table
    public DbSet<Customer> Customers { get; set; }   // → "Customers" table
    public DbSet<Order> Orders { get; set; }          // → "Orders" table
    public DbSet<Product> Products { get; set; }      // → "Products" table
}
```

- `DbSet<T>` is EF Core's representation of a **table**. It implements `IQueryable<T>`, so you can write LINQ queries against it.
- Each `DbSet<T>` property tells EF Core: "This entity type has a corresponding table in the database."

---

## Configuring the Connection

There are two main ways to tell `DbContext` which database to connect to.

### Option 1: Override `OnConfiguring` (simple / console apps)

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Customer> Customers { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // Connection string directly in code (fine for learning, not for production)
        optionsBuilder.UseSqlServer(
            "Server=.;Database=MyApp;Trusted_Connection=true;TrustServerCertificate=true;");
    }
}
```

### Option 2: Dependency Injection (ASP.NET Core / production apps)

```csharp
// In Program.cs (or Startup.cs)
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
```

```json
// appsettings.json
{
  "ConnectionStrings": {
    "Default": "Server=.;Database=MyApp;Trusted_Connection=true;TrustServerCertificate=true;"
  }
}
```

```csharp
// The DbContext receives options through its constructor
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)   // pass options to the base DbContext
    { }

    public DbSet<Customer> Customers { get; set; }
}
```

```ad-tip
**Always use the constructor + DI approach in production**. It keeps connection strings out of your code, makes testing easier (you can swap in a test database), and lets the DI container manage the lifetime of the `DbContext`.
```

### Why DbContextOptions Exists

`DbContextOptions` decouples the DbContext from **how** it connects to a database. The same DbContext class can work with different providers, connection strings, or configurations — you just pass different options:

```csharp
// Production — SQL Server
var prodOptions = new DbContextOptionsBuilder<AppDbContext>()
    .UseSqlServer(prodConnectionString)
    .Options;
using var prodDb = new AppDbContext(prodOptions);

// Testing — SQLite in-memory
var testOptions = new DbContextOptionsBuilder<AppDbContext>()
    .UseSqlite("Data Source=:memory:")
    .Options;
using var testDb = new AppDbContext(testOptions);

// Same AppDbContext class, different databases
```

The DbContext doesn't know or care which database it's talking to — that decision is made by whoever creates the options. This is what makes it possible to swap providers at runtime (e.g., MySQL in development, SQL Server in production) without changing any of your entity or query code. See [[Database Providers]] for provider details.

---

## Connection String Configuration

- A **connection string** tells EF Core *where* the database is, *which* database to use, and *how* to authenticate.
- Common connection string for SQL Server:

```
Server=.;Database=MyApp;Trusted_Connection=true;TrustServerCertificate=true;
```

| Part                       | Meaning                                                |
| -------------------------- | ------------------------------------------------------ |
| `Server=.`                 | Local machine (`.` = localhost for SQL Server)          |
| `Database=MyApp`           | Database name                                          |
| `Trusted_Connection=true`  | Use Windows Authentication (no username/password)      |
| `TrustServerCertificate=true` | Skip SSL cert validation (development only)         |

- For SQL Authentication (username/password):

```
Server=myserver.database.windows.net;Database=MyApp;User Id=sa;Password=YourPassword;
```

- For more, see [[Connection Strings]].

---

## DbContext Lifetime: Create, Use, Dispose

- `DbContext` is designed to be **short-lived**. Create it, do your work, then dispose it.
- It is **not thread-safe** — never share a single `DbContext` across threads or concurrent requests.

### Using Statement Pattern (console / desktop apps)

```csharp
// DbContext implements IDisposable — always dispose it
using (var db = new AppDbContext())
{
    var customers = db.Customers
        .Where(c => c.City == "Seattle")
        .ToList();

    // Modify and save
    customers[0].City = "Portland";
    db.SaveChanges();
}
// Connection is closed, change tracker is cleared
```

Or with the C# 8+ using declaration:

```csharp
using var db = new AppDbContext();

db.Customers.Add(new Customer { Name = "Alice", City = "Denver" });
db.SaveChanges();
// Disposed at end of scope
```

### DI-Managed Lifetime (ASP.NET Core)

- When using `AddDbContext<T>()`, the DI container creates a **new `DbContext` per HTTP request** (Scoped lifetime) and disposes it when the request ends.
- You don't call `Dispose()` yourself — the framework handles it.

```csharp
public class CustomersController : ControllerBase
{
    private readonly AppDbContext _db;

    // DI injects a fresh DbContext for each request
    public CustomersController(AppDbContext db)
    {
        _db = db;
    }

    [HttpGet]
    public IActionResult GetAll()
    {
        return Ok(_db.Customers.ToList());
    }
}
```

```ad-warning
**Never register `DbContext` as Singleton**. A singleton `DbContext` lives for the entire app lifetime, accumulates tracked entities in memory (memory leak), and will cause concurrency bugs because it is not thread-safe.

The correct lifetime is **Scoped** (one per request), which is the default when using `AddDbContext`.
```

---

## What DbContext Does When You Call SaveChanges

1. **Detects changes** — scans all tracked entities and compares current property values to their original (snapshot) values
2. **Generates SQL** — creates INSERT, UPDATE, or DELETE statements for each change
3. **Opens a transaction** — wraps all SQL in an implicit `DbTransaction`
4. **Executes SQL** — sends each statement via `DbCommand` (ADO.NET)
5. **Commits** — if all statements succeed, the transaction is committed
6. **Updates entity state** — sets Added entities to Unchanged, applies database-generated values (like auto-increment IDs) back onto the objects

```csharp
var customer = new Customer { Name = "Bob" };  // Id = 0 (not yet in DB)
db.Customers.Add(customer);                    // State: Added
db.SaveChanges();                              // INSERT INTO Customers ...
// customer.Id is now populated (e.g., 42) with the DB-generated value
```

---

## OnModelCreating — Fluent Configuration

- `DbContext` has a virtual method `OnModelCreating` where you can configure the model using the **Fluent API**.
- This is where you configure table names, column types, relationships, indexes, and seed data.

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Order> Orders { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure table name explicitly
        modelBuilder.Entity<Customer>().ToTable("Customers");

        // Configure a property
        modelBuilder.Entity<Customer>()
            .Property(c => c.Name)
            .HasMaxLength(100)
            .IsRequired();

        // Configure a relationship
        modelBuilder.Entity<Order>()
            .HasOne(o => o.Customer)
            .WithMany(c => c.Orders)
            .HasForeignKey(o => o.CustomerId);
    }
}
```

- For details on entity configuration, see [[Entity Classes]].
- For relationship configuration, see [[Relationships]].

---

## See Also

- [[EF Core Overview]] — where DbContext fits in the EF Core architecture
- [[Entity Classes]] — the POCO classes that DbContext manages
- [[Relationships]] — configuring navigation properties and foreign keys
- [[Database Providers]] — how DbContext connects to different databases
- [[Connection Strings]] — connection string formats for different databases
- [[DbConnection]] — the ADO.NET connection that DbContext wraps
