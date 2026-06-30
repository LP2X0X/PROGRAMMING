---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - lifetimes
---

## Transient

**Transient** services are created every single time they are requested from the container. No caching, no reuse -- every injection point gets a brand-new instance.

### Registration

```csharp
builder.Services.AddTransient<IOrderValidator, OrderValidator>();
```

### Behavior

Even within the same HTTP request, if two different classes both depend on `IOrderValidator`, they each receive a **different** instance:

```csharp
public class OrderController : ControllerBase
{
    private readonly IOrderValidator _validator; // Instance A

    public OrderController(IOrderValidator validator)
    {
        _validator = validator;
    }
}

public class OrderService
{
    private readonly IOrderValidator _validator; // Instance B (different from A!)

    public OrderService(IOrderValidator validator)
    {
        _validator = validator;
    }
}
```

If both `OrderController` and `OrderService` are resolved during the same request, `_validator` in each class points to a **separate** `OrderValidator` instance.

### When to Use Transient

Transient is ideal for:
- **Lightweight, stateless services** -- validators, formatters, mapping services
- **Services with no shared mutable state** -- each consumer gets its own clean copy
- **Services that are cheap to construct** -- since a new one is created every time

```csharp
// Good candidates for Transient
builder.Services.AddTransient<IOrderValidator, OrderValidator>();
builder.Services.AddTransient<IAddressFormatter, AddressFormatter>();
builder.Services.AddTransient<ICustomerMapper, CustomerMapper>();
```

> [!warning] Common Misconception
> "Transient means one instance per request." This is wrong. Transient means one instance **per injection**. A single request that injects the same transient service in three places will create three separate instances. If you want one-per-request, use **Scoped**.

> [!ad-note]
> Transient services that implement `IDisposable` are tracked by the container and disposed when the scope (request) ends. This means transient disposable services still have their `Dispose()` called -- but many transient instances may accumulate within a single request, increasing memory pressure.

> [!summary] Section Summary
> - Transient creates a new instance for every injection point, every time.
> - Even within the same HTTP request, different constructor parameters get different instances.
> - Best suited for lightweight, stateless, cheap-to-create services.
> - Disposable transient services are tracked and disposed at scope end, but many instances can accumulate.
> - Do not confuse Transient with "one per request" -- that is Scoped.
