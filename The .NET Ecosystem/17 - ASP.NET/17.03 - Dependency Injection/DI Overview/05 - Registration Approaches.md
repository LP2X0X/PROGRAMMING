---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - fundamentals
---

## Registration Approaches

There are three main ways to register services with the DI container. Each serves a different purpose.

### Interface-Based Registration

The most common and recommended approach. You map an interface (the service type) to a concrete class (the implementation type).

```csharp
// "When someone asks for IOrderService, give them an OrderService"
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<IInventoryRepository, SqlInventoryRepository>();
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
```

This is the approach that enables the Dependency Inversion Principle -- consuming classes depend only on the interface, never on the concrete type.

> [!tip] When to Use Interface-Based Registration
> Use this as your default approach. It allows you to:
> - Swap implementations without changing consuming code
> - Mock dependencies in unit tests
> - Follow SOLID principles

### Self-Registration (Concrete Type Only)

You can register a concrete class without an interface. The service type and implementation type are the same.

```csharp
// "When someone asks for OrderService (the concrete class), create one"
builder.Services.AddScoped<OrderService>();

// Equivalent to:
builder.Services.AddScoped<OrderService, OrderService>();
```

```csharp
// Usage -- the controller depends on the concrete type directly
public class OrderController : ControllerBase
{
    private readonly OrderService _orderService;

    public OrderController(OrderService orderService)
    {
        _orderService = orderService;
    }
}
```

> [!ad-note]
> Self-registration is useful for classes that are unlikely to have multiple implementations, such as configuration objects, helper classes, or application-specific services with no foreseeable need for substitution. However, it does make unit testing harder since you cannot easily substitute a mock without an interface.

### Factory-Based Registration

Factory registration gives you full control over how an instance is created. You provide a lambda that receives the `IServiceProvider` and returns the service instance.

```csharp
// Simple factory
builder.Services.AddScoped<IOrderService>(serviceProvider =>
{
    var inventory = serviceProvider.GetRequiredService<IInventoryRepository>();
    var payment = serviceProvider.GetRequiredService<IPaymentGateway>();
    var email = serviceProvider.GetRequiredService<IEmailService>();
    var logger = serviceProvider.GetRequiredService<ILogger<OrderService>>();

    return new OrderService(inventory, payment, email, logger);
});
```

```csharp
// Factory with conditional logic
builder.Services.AddScoped<IPaymentGateway>(serviceProvider =>
{
    var config = serviceProvider.GetRequiredService<IConfiguration>();
    var environment = config["PaymentEnvironment"];

    return environment switch
    {
        "production" => new StripePaymentGateway(config["Stripe:LiveKey"]!),
        "sandbox" => new StripePaymentGateway(config["Stripe:TestKey"]!),
        _ => new FakePaymentGateway() // For local development
    };
});
```

```csharp
// Factory for classes that need special initialization
builder.Services.AddSingleton<IInventoryCache>(serviceProvider =>
{
    var logger = serviceProvider.GetRequiredService<ILogger<InventoryCache>>();
    var cache = new InventoryCache(logger);
    cache.Preload(); // Custom initialization step
    return cache;
});
```

> [!tip] When to Use Factory Registration
> Use factories when:
> - The constructor needs values that are not themselves registered services (e.g., configuration strings, computed values)
> - You need conditional logic to decide which implementation to return
> - The object requires custom initialization steps beyond what the constructor does
> - You are integrating with third-party classes whose constructors you do not control

### Comparison Table

| Approach | Syntax | Best For |
|---|---|---|
| Interface-based | `AddScoped<IService, Implementation>()` | Most services -- enables substitution and testing |
| Self-registration | `AddScoped<ConcreteClass>()` | Simple classes with no need for abstraction |
| Factory-based | `AddScoped<IService>(sp => ...)` | Complex creation logic, conditional implementations |

> [!summary] Section Summary
> - **Interface-based** (`AddScoped<IService, Implementation>()`) is the default and recommended approach for most services.
> - **Self-registration** (`AddScoped<ConcreteClass>()`) registers a concrete type directly, useful for classes that will not be substituted.
> - **Factory-based** (`AddScoped<IService>(sp => ...)`) provides a lambda for custom construction logic, conditional implementations, or special initialization.
> - Interface-based registration best supports SOLID principles, testability, and implementation swapping.
> - Factory registration is the escape hatch for scenarios where simple type mapping is not enough.
