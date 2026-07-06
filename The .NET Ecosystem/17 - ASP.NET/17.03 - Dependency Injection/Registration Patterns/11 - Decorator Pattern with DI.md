---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
---

## Decorator Pattern with DI

The decorator pattern wraps an existing service with additional behavior (logging, caching, retry logic) without modifying the original implementation.

### The problem

ASP.NET Core's built-in container does not natively support decorators. You cannot simply register two classes for the same interface and have one wrap the other automatically.

### Manual approach with factory registration

```csharp
public interface IOrderService
{
    Task<Order> PlaceOrderAsync(OrderRequest request);
}

public class OrderService : IOrderService
{
    private readonly IOrderRepository _repository;
    private readonly IInventoryService _inventory;

    public OrderService(IOrderRepository repository, IInventoryService inventory)
    {
        _repository = repository;
        _inventory = inventory;
    }

    public async Task<Order> PlaceOrderAsync(OrderRequest request)
    {
        // Core order placement logic
        var order = new Order(request);
        await _inventory.ReserveItemsAsync(order.Items);
        await _repository.SaveAsync(order);
        return order;
    }
}

public class LoggingOrderService : IOrderService
{
    private readonly IOrderService _inner;
    private readonly ILogger<LoggingOrderService> _logger;

    public LoggingOrderService(IOrderService inner, ILogger<LoggingOrderService> logger)
    {
        _inner = inner;
        _logger = logger;
    }

    public async Task<Order> PlaceOrderAsync(OrderRequest request)
    {
        _logger.LogInformation("Placing order for customer {CustomerId}", request.CustomerId);
        var stopwatch = Stopwatch.StartNew();

        var order = await _inner.PlaceOrderAsync(request);

        stopwatch.Stop();
        _logger.LogInformation(
            "Order {OrderId} placed in {ElapsedMs}ms",
            order.Id, stopwatch.ElapsedMilliseconds);

        return order;
    }
}
```

### Registration using a factory delegate

```csharp
builder.Services.AddScoped<OrderService>();
builder.Services.AddScoped<IOrderService>(sp =>
{
    var inner = sp.GetRequiredService<OrderService>();
    var logger = sp.GetRequiredService<ILogger<LoggingOrderService>>();
    return new LoggingOrderService(inner, logger);
});
```

> [!ad-note]
> Notice that `OrderService` is registered as itself (self-registration), while `IOrderService` is registered via a factory that wraps `OrderService` with `LoggingOrderService`. This avoids infinite recursion -- if both were registered as `IOrderService`, resolving `IOrderService` inside the factory would create an infinite loop.

### Stacking multiple decorators

```csharp
builder.Services.AddScoped<OrderService>();

builder.Services.AddScoped<IOrderService>(sp =>
{
    var inner = sp.GetRequiredService<OrderService>();

    // First decorator: logging
    var loggingDecorator = new LoggingOrderService(
        inner,
        sp.GetRequiredService<ILogger<LoggingOrderService>>());

    // Second decorator: caching
    var cachingDecorator = new CachingOrderService(
        loggingDecorator,
        sp.GetRequiredService<IMemoryCache>());

    return cachingDecorator;
});
```

### Using Scrutor for automatic decoration

The third-party library **Scrutor** adds a `Decorate` method that simplifies this pattern significantly:

```csharp
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.Decorate<IOrderService, LoggingOrderService>();
builder.Services.Decorate<IOrderService, CachingOrderService>();
```

> [!tip]
> If you find yourself writing multiple decorator registrations with factory delegates, Scrutor is worth the dependency. It handles the resolution chain correctly and keeps your registration code clean.

> [!summary] Section Summary
> - The built-in DI container does not natively support the decorator pattern.
> - Use a factory delegate: register the inner service as itself, then register the interface with a factory that wraps it.
> - Register the **inner** service by its concrete type to avoid infinite recursion when resolving.
> - Multiple decorators can be stacked by nesting them in the factory delegate.
> - Scrutor (third-party NuGet package) adds `Decorate<TService, TDecorator>()` for clean decorator registration.
