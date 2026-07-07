---
tags:
  - csharp
  - asp-net-core
  - web-api
  - data-access
---

A **Data Access Object (DAO)** is a class that encapsulates all database operations (CRUD) for a specific entity. It separates *how* data is stored from the rest of the application.

### DAO vs DTO

| | DTO | DAO |
|---|---|---|
| **Purpose** | Carries data between layers | Handles database operations |
| **Contains** | Properties only | Methods (CRUD logic) |
| **Example** | `ProductDto` | `ProductRepository` |

```
Client ←→ [DTO] ←→ Controller ←→ Service ←→ [DAO] ←→ Database
              ↑ just data                        ↑ reads/writes data
```

### DAO in .NET — The Repository Pattern

In .NET/EF Core, the DAO pattern is commonly called the **Repository pattern**. Same concept, different name:

```csharp
// The DAO / Repository — encapsulates data access
public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    public Product? GetById(int id)
        => _context.Products.Find(id);

    public IEnumerable<Product> GetAll()
        => _context.Products.ToList();

    public void Add(Product product)
        => _context.Products.Add(product);

    public void Update(Product product)
        => _context.Products.Update(product);

    public void Delete(Product product)
        => _context.Products.Remove(product);

    public void Save()
        => _context.SaveChanges();
}
```

The interface:

```csharp
public interface IProductRepository
{
    Product? GetById(int id);
    IEnumerable<Product> GetAll();
    void Add(Product product);
    void Update(Product product);
    void Delete(Product product);
    void Save();
}
```

### Why Use a DAO/Repository?

1. **Swappable storage** — change from SQL Server to MongoDB without touching controllers
2. **Testable** — mock the interface in unit tests instead of hitting a real database
3. **Single responsibility** — controllers handle HTTP, repositories handle data
4. **Reusable** — multiple services can share the same repository

### Do You Always Need One with EF Core?

EF Core's `DbContext` is already a DAO of sorts — it wraps data access behind `DbSet<T>`. Adding a repository on top is an extra layer that not everyone agrees is necessary:

- **Use it** when you want to abstract away EF Core, add caching, or have complex queries worth encapsulating
- **Skip it** for simple CRUD apps where the controller can use `DbContext` directly

> [!summary] Section Summary
> A DAO (Data Access Object) encapsulates database operations behind an interface. In .NET, this is typically implemented as the Repository pattern. It pairs with [[07 - Data Transfer Objects (DTOs)|DTOs]] — DTOs carry the data, DAOs read and write it.
