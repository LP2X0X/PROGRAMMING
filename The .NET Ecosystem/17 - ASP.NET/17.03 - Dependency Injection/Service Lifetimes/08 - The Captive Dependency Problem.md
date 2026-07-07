---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - lifetimes
---

## The Captive Dependency Problem

The **captive dependency** (also called **captured dependency**) is one of the most insidious DI bugs. It occurs when a longer-lived service holds a reference to a shorter-lived service, causing the shorter-lived service to live far beyond its intended lifetime.

### The Dangerous Code

```csharp
// Registration
builder.Services.AddSingleton<IOrderNotificationService, OrderNotificationService>();
builder.Services.AddScoped<IOrderRepository, OrderRepository>(); // Uses DbContext internally

// The problematic service
public class OrderNotificationService : IOrderNotificationService
{
    private readonly IOrderRepository _orderRepo; // CAPTIVE! Scoped inside a Singleton

    public OrderNotificationService(IOrderRepository orderRepo)
    {
        _orderRepo = orderRepo;
    }

    public async Task NotifyOrderShippedAsync(int orderId)
    {
        // This repository (and its DbContext) was created once when the
        // singleton was first resolved. It is now stale and will be used
        // for ALL future requests -- it will never be disposed or refreshed.
        var order = await _orderRepo.GetByIdAsync(orderId);
        // ... send notification
    }
}
```

### Why This Is Dangerous

1. **Stale DbContext.** The `OrderRepository` (and its `DbContext`) was resolved once when the singleton was created. It holds onto the same database connection and change tracker forever.
2. **Memory leak.** The change tracker accumulates every entity ever loaded, growing without bound.
3. **Connection exhaustion.** The held connection may time out, be reclaimed by the pool, or become stale -- leading to `SqlException` at runtime.
4. **Thread safety violation.** The singleton is accessed by multiple concurrent requests, but the scoped `DbContext` inside it is not thread-safe.
5. **No per-request isolation.** The scoped service was designed to be fresh per request, but as a captive, it is shared across all requests.

> [!danger]
> The captive dependency problem is especially treacherous because it often works correctly in development (single user, low traffic) and only fails under production load or after the application has been running for a while.

### The Fix: Use IServiceScopeFactory

The correct approach is for the singleton to create a new scope each time it needs to access a scoped service:

```csharp
public class OrderNotificationService : IOrderNotificationService
{
    private readonly IServiceScopeFactory _scopeFactory;

    public OrderNotificationService(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }

    public async Task NotifyOrderShippedAsync(int orderId)
    {
        // Create a fresh scope -- gets a fresh DbContext and repository
        using var scope = _scopeFactory.CreateScope();
        var orderRepo = scope.ServiceProvider
            .GetRequiredService<IOrderRepository>();

        var order = await orderRepo.GetByIdAsync(orderId);
        // ... send notification
    }
    // Scope disposed here -- DbContext cleaned up properly
}
```

> [!tip]
> The rule is simple: **services can only depend on services with an equal or longer lifetime.**
> - Singleton can depend on: Singleton
> - Scoped can depend on: Scoped, Singleton
> - Transient can depend on: Transient, Scoped, Singleton
>
> If a singleton needs a scoped service, use `IServiceScopeFactory` to create a scope on demand.

> [!ad-note]
> The same problem applies to a Scoped service holding a Transient dependency. The transient instance is captured for the lifetime of the scope (the full request), which is usually acceptable but may not be if the transient was designed to be truly ephemeral. In practice, the Singleton-captures-Scoped case is by far the most common and dangerous.

See also: [[Common DI Pitfalls]] for more captive dependency scenarios.

> [!summary] Section Summary
> - A captive dependency occurs when a longer-lived service captures a shorter-lived service.
> - The most dangerous case: a Singleton holding a Scoped service (like DbContext).
> - Symptoms include stale data, memory leaks, connection exhaustion, and thread-safety violations.
> - The fix is to inject `IServiceScopeFactory` and create a new scope for each operation.
> - The lifetime rule: services can only depend on equal or longer-lived services.
