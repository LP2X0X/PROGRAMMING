---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - pitfalls
---

## Service Locator Anti-Pattern

> [!info] Definition
> The **Service Locator** anti-pattern occurs when a class takes a dependency on `IServiceProvider` and manually resolves its dependencies using `GetService<T>()` or `GetRequiredService<T>()`, instead of declaring its dependencies explicitly through constructor injection.

### The Anti-Pattern

```csharp
// OrderProcessingService.cs -- SERVICE LOCATOR ANTI-PATTERN
public class OrderProcessingService : IOrderProcessingService
{
    private readonly IServiceProvider _serviceProvider;

    // The only declared dependency is the service locator itself.
    // What does this class ACTUALLY need? You cannot tell from the constructor.
    public OrderProcessingService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public async Task ProcessOrderAsync(Order order)
    {
        // Hidden dependencies -- resolved at runtime, invisible to the consumer
        var inventoryService = _serviceProvider.GetRequiredService<IInventoryService>();
        var paymentGateway = _serviceProvider.GetRequiredService<IPaymentGateway>();
        var emailService = _serviceProvider.GetRequiredService<IEmailService>();
        var auditLogger = _serviceProvider.GetRequiredService<IAuditLogger>();

        await inventoryService.ReserveStockAsync(order.Items);
        await paymentGateway.ChargeAsync(order.Total);
        await emailService.SendOrderConfirmationAsync(order);
        await auditLogger.LogAsync($"Order {order.Id} processed");
    }
}
```

### Why It Is Bad

- **Hidden dependencies**: The constructor signature `(IServiceProvider)` tells you nothing about what the class actually needs. You must read the entire implementation to discover its real dependencies.
- **Testing is painful**: In unit tests, you must set up a mock `IServiceProvider` that returns mocks for every service the class resolves internally. If the implementation adds a new `GetRequiredService<T>()` call, existing tests break with no compiler warning.
- **No compile-time safety**: If `IInventoryService` is not registered, you only find out at runtime when `GetRequiredService` throws -- and only when the specific code path is hit.
- **Defeats the purpose of DI**: The whole point of DI is to make dependencies explicit, visible, and substitutable. Service Locator inverts this back to opaque, hidden resolution.

### The Correct Approach

```csharp
// OrderProcessingService.cs -- PROPER CONSTRUCTOR INJECTION
public class OrderProcessingService : IOrderProcessingService
{
    private readonly IInventoryService _inventoryService;
    private readonly IPaymentGateway _paymentGateway;
    private readonly IEmailService _emailService;
    private readonly IAuditLogger _auditLogger;

    // All dependencies are explicit and visible.
    // You know exactly what this class needs by looking at its constructor.
    public OrderProcessingService(
        IInventoryService inventoryService,
        IPaymentGateway paymentGateway,
        IEmailService emailService,
        IAuditLogger auditLogger)
    {
        _inventoryService = inventoryService;
        _paymentGateway = paymentGateway;
        _emailService = emailService;
        _auditLogger = auditLogger;
    }

    public async Task ProcessOrderAsync(Order order)
    {
        await _inventoryService.ReserveStockAsync(order.Items);
        await _paymentGateway.ChargeAsync(order.Total);
        await _emailService.SendOrderConfirmationAsync(order);
        await _auditLogger.LogAsync($"Order {order.Id} processed");
    }
}
```

### When Service Locator IS Acceptable

There are legitimate scenarios where resolving from `IServiceProvider` is the right choice:

**1. Factory patterns where the type is determined at runtime:**

```csharp
public class NotificationDispatcher
{
    private readonly IServiceProvider _serviceProvider;

    public NotificationDispatcher(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public async Task DispatchAsync(Notification notification)
    {
        // The concrete handler type is not known until runtime.
        // This is a valid use of IServiceProvider.
        var handlerType = typeof(INotificationHandler<>)
            .MakeGenericType(notification.GetType());

        var handler = _serviceProvider.GetRequiredService(handlerType);

        await ((dynamic)handler).HandleAsync(notification);
    }
}
```

**2. Middleware that needs Scoped services:**

```csharp
// In middleware, scoped services must be resolved from HttpContext.RequestServices,
// which is essentially using a service locator -- but this is by design.
public class TenantMiddleware
{
    private readonly RequestDelegate _next;

    public TenantMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Resolving from the request scope -- acceptable in middleware
        var tenantService = context.RequestServices
            .GetRequiredService<ITenantService>();

        tenantService.SetCurrentTenant(context.Request.Headers["X-Tenant-Id"]);
        await _next(context);
    }
}
```

**3. Background services that need to create scopes** (as shown in the [[Captive Dependency (Lifestyle Mismatch)]] section with `IServiceScopeFactory`).

> [!tip]
> The key distinction: using `IServiceProvider` is acceptable when the **type** to resolve is not known at compile time, or when you are in infrastructure code (middleware, hosted services) that must bridge between DI scopes. It is an anti-pattern when used to hide dependencies that are perfectly well known at compile time.

> [!summary] Section Summary
> - Service Locator hides dependencies behind `IServiceProvider`, making code harder to understand and test
> - Constructor injection makes all dependencies explicit, visible, and verifiable at compile time
> - Service Locator is acceptable for runtime-determined types, middleware scope bridging, and background service scoping
> - If you know the concrete interface you need at compile time, always prefer constructor injection
> - A class taking `IServiceProvider` as its only dependency is a strong code smell
