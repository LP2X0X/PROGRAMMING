---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
---

## Open Generics

Open generic registration lets you register a single mapping that covers **all closed generic forms** of an interface.

### The generic interface and implementation

```csharp
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IReadOnlyList<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

public class SqlRepository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _context;

    public SqlRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<T?> GetByIdAsync(int id)
        => await _context.Set<T>().FindAsync(id);

    public async Task<IReadOnlyList<T>> GetAllAsync()
        => await _context.Set<T>().ToListAsync();

    public async Task AddAsync(T entity)
        => await _context.Set<T>().AddAsync(entity);

    public async Task UpdateAsync(T entity)
        => _context.Set<T>().Update(entity);

    public async Task DeleteAsync(int id)
    {
        var entity = await GetByIdAsync(id);
        if (entity is not null)
            _context.Set<T>().Remove(entity);
    }
}
```

### Registration using `typeof`

```csharp
// One registration covers IRepository<Order>, IRepository<Customer>,
// IRepository<Product>, and every other entity type
builder.Services.AddScoped(typeof(IRepository<>), typeof(SqlRepository<>));
```

### Usage -- the container fills in the type argument automatically

```csharp
public class CustomerController : ControllerBase
{
    private readonly IRepository<Customer> _customers;
    private readonly IRepository<Order> _orders;

    public CustomerController(
        IRepository<Customer> customers,
        IRepository<Order> orders)
    {
        _customers = customers;
        _orders = orders;
    }
}
```

> [!warning] Common Misconception
> You might wonder why you cannot write `builder.Services.AddScoped<IRepository<>, SqlRepository<>>()`. The generic extension method form requires **closed** generic types (e.g., `IRepository<Order>`). Open generics (with empty angle brackets `<>`) are only representable via `typeof()`, so you must use the non-generic `AddScoped(Type, Type)` overload.

> [!ad-note]
> You can combine open generics with specific overrides. Register the open generic first, then register specific closed-generic versions for types that need special handling. The container prefers the specific registration:
>
> ```csharp
> builder.Services.AddScoped(typeof(IRepository<>), typeof(SqlRepository<>));
> builder.Services.AddScoped<IRepository<AuditLog>, ReadOnlyAuditRepository>();
> ```
>
> Now `IRepository<Customer>` resolves to `SqlRepository<Customer>`, but `IRepository<AuditLog>` resolves to `ReadOnlyAuditRepository`.

> [!summary] Section Summary
> - Open generic registration uses `Add{Lifetime}(typeof(IService<>), typeof(Implementation<>))`.
> - A single registration covers every closed form of the generic type.
> - The generic extension method syntax (`Add<T1, T2>`) cannot be used with open generics -- use `typeof`.
> - You can override specific closed-generic types after registering the open generic.
> - This is the standard pattern for generic repository, handler, and pipeline abstractions.
