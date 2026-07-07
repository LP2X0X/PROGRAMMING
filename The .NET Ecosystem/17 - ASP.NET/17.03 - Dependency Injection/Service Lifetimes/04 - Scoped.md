---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - lifetimes
---

## Scoped

**Scoped** services are created once per scope. In ASP.NET Core, each HTTP request automatically gets its own scope, so scoped services are effectively **one instance per HTTP request**.

### Registration

```csharp
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
```

### Behavior

Within a single request, every class that asks for `IOrderRepository` gets the **same** instance:

```csharp
public class OrderController : ControllerBase
{
    private readonly IOrderRepository _repo; // Instance A

    public OrderController(IOrderRepository repo)
    {
        _repo = repo;
    }
}

public class OrderService
{
    private readonly IOrderRepository _repo; // Also Instance A (same!)

    public OrderService(IOrderRepository repo)
    {
        _repo = repo;
    }
}
```

But a **different** HTTP request creates a completely new scope and therefore a new instance:

```
Request 1: OrderRepository instance #1 (shared within request 1)
Request 2: OrderRepository instance #2 (shared within request 2)
Request 3: OrderRepository instance #3 (shared within request 3)
```

### When to Use Scoped

Scoped is ideal for:
- **EF Core DbContext** -- the most common scoped service
- **Unit of Work** implementations
- **Request-specific state** -- anything tied to the current user or operation
- **Services that need to share state within a single request** but be isolated between requests

```csharp
// Good candidates for Scoped
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<ICurrentUserService, CurrentUserService>();
```

> [!tip]
> Scoped is the "default safe choice" for most business services. If you are unsure, start with Scoped -- it prevents cross-request data leaks (unlike Singleton) while avoiding unnecessary instance creation (unlike Transient).

> [!ad-note]
> Scoped services are disposed at the end of the request when the scope is disposed. This is why `DbContext` works well as scoped -- its connections and change tracker are cleaned up automatically after each request.

> [!summary] Section Summary
> - Scoped creates one instance per scope (one per HTTP request in ASP.NET Core).
> - All classes within the same request share the same scoped instance.
> - Different requests always get different instances.
> - Ideal for DbContext, Unit of Work, and request-specific state.
> - Scoped services are disposed when the scope (request) ends.
