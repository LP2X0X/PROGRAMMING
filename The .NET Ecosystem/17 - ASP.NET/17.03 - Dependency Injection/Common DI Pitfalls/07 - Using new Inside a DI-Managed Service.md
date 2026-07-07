---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - pitfalls
---

## Using `new` Inside a DI-Managed Service

> [!info] Definition
> When a DI-managed service creates its dependencies using `new` instead of receiving them through constructor injection, it defeats the entire purpose of the DI system. The manually created instance does not participate in the DI lifecycle and cannot have its own dependencies injected.

### The Problem

```csharp
// OrderService.cs -- USING new DEFEATS DI
public class OrderService : IOrderService
{
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        // Creating the repository manually with 'new'.
        // But OrderRepository needs a DbContext in its constructor!
        // This either won't compile or requires a parameterless constructor
        // that doesn't use the shared DbContext.
        var repository = new OrderRepository(); // BAD

        // Even if it compiles, the repository is:
        // - Not using the container-managed DbContext
        // - Not participating in the request scope
        // - Not disposable by the container
        // - Impossible to swap out for testing

        var order = new Order(request.CustomerId, request.Items);
        await repository.SaveAsync(order);
        return order;
    }
}
```

```csharp
// The repository expects a DbContext from DI
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _dbContext;

    public OrderRepository(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public async Task SaveAsync(Order order)
    {
        _dbContext.Orders.Add(order);
        await _dbContext.SaveChangesAsync();
    }
}
```

The problems:

- `new OrderRepository()` will not compile because `OrderRepository` requires an `AppDbContext` parameter
- Even if you somehow provide a `DbContext`, it is not the one managed by the container -- transactions, change tracking, and connection pooling all break
- You cannot substitute a mock `IOrderRepository` in unit tests because the dependency is hardcoded
- If `OrderRepository` has its own dependencies (logging, configuration, etc.), those are also missing

### The Fix

```csharp
// OrderService.cs -- PROPER INJECTION
public class OrderService : IOrderService
{
    private readonly IOrderRepository _repository;

    // Let the container provide the repository with all its dependencies wired up
    public OrderService(IOrderRepository repository)
    {
        _repository = repository;
    }

    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        var order = new Order(request.CustomerId, request.Items);
        await _repository.SaveAsync(order);
        return order;
    }
}
```

```csharp
// Registration in Program.cs
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddScoped<IOrderService, OrderService>();
```

> [!ad-note]
> Using `new` is perfectly fine for data objects, DTOs, value objects, and simple models like `new Order(...)` or `new CreateOrderRequest()`. The rule only applies to **services** -- classes that have behavior and their own dependencies. See [[Registration Patterns]] for guidelines on what should and should not be registered in the container.

> [!summary] Section Summary
> - Using `new` to create service dependencies bypasses the DI container entirely
> - The manually created instance does not get its own dependencies injected (DbContext, loggers, etc.)
> - It creates tight coupling, prevents testing with mocks, and breaks lifecycle management
> - Always inject service dependencies through the constructor and register them in `Program.cs`
> - Using `new` for data objects (DTOs, models, value objects) is perfectly fine -- only service creation is the concern
