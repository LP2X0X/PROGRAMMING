---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
---

## Basic Registration

This is the most common registration pattern in ASP.NET Core. You map an interface (the **service type**) to a concrete class (the **implementation type**) with one of three lifetime methods.

> [!info] Definition
> **Basic registration** uses the `Add{Lifetime}<TService, TImplementation>()` extension methods where `TService` is typically an interface and `TImplementation` is the concrete class that implements it.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Transient: new instance every time it is requested
builder.Services.AddTransient<IOrderValidator, OrderValidator>();

// Scoped: one instance per HTTP request (or per scope)
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();

// Singleton: one instance for the entire application lifetime
builder.Services.AddSingleton<ICurrencyConverter, CurrencyConverter>();
```

When the container encounters a constructor parameter of type `IOrderRepository`, it creates (or reuses, depending on the lifetime) an instance of `SqlOrderRepository` and injects it.

```csharp
public class OrderController : ControllerBase
{
    private readonly IOrderRepository _orderRepository;
    private readonly IOrderValidator _validator;

    // Both dependencies resolved automatically by the container
    public OrderController(IOrderRepository orderRepository, IOrderValidator validator)
    {
        _orderRepository = orderRepository;
        _validator = validator;
    }
}
```

> [!tip]
> When in doubt, start with **scoped** for anything that touches a database or per-request state, **singleton** for stateless utilities, and **transient** for lightweight, stateless services that hold no shared state. See [[Service Lifetimes]] for the full decision framework.

> [!summary] Section Summary
> - Basic registration maps an interface to a concrete class using `Add{Lifetime}<TService, TImpl>()`.
> - `AddTransient` creates a new instance on every resolution, `AddScoped` creates one per scope/request, and `AddSingleton` creates one for the app's lifetime.
> - This is the pattern you will use for the vast majority of your registrations.
> - The container automatically resolves constructor parameters by their service type.
