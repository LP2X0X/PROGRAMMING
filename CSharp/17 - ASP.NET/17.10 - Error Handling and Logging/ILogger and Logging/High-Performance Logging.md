---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---


For hot paths (code that executes thousands of times per second), the overhead of parsing message templates and boxing value-type arguments on every call can become measurable. .NET provides **`LoggerMessage.Define`** for zero-allocation, pre-compiled log delegates.

## LoggerMessage.Define (Pre-.NET 8)

```csharp
public partial class OrderService
{
    // Pre-compiled log delegates -- message template is parsed once
    private static readonly Action<ILogger, int, string, Exception?> _orderCreated =
        LoggerMessage.Define<int, string>(
            LogLevel.Information,
            new EventId(1001, nameof(OrderCreated)),
            "Order {OrderId} created for customer {CustomerName}");

    private static readonly Action<ILogger, int, Exception?> _orderFailed =
        LoggerMessage.Define<int>(
            LogLevel.Error,
            new EventId(1002, nameof(OrderFailed)),
            "Failed to create order for customer {CustomerId}");

    private void OrderCreated(int orderId, string customerName)
        => _orderCreated(_logger, orderId, customerName, null);

    private void OrderFailed(int customerId, Exception ex)
        => _orderFailed(_logger, customerId, ex);
}
```

## Source Generators (C# 12 / .NET 8+)

.NET 8 introduced the **`[LoggerMessage]` attribute** with source generators, which is much cleaner:

```csharp
public partial class OrderService
{
    private readonly ILogger<OrderService> _logger;

    [LoggerMessage(
        EventId = 1001,
        Level = LogLevel.Information,
        Message = "Order {OrderId} created for customer {CustomerName}")]
    partial void LogOrderCreated(int orderId, string customerName);

    [LoggerMessage(
        EventId = 1002,
        Level = LogLevel.Error,
        Message = "Failed to create order for customer {CustomerId}")]
    partial void LogOrderFailed(int customerId, Exception ex);

    public async Task CreateOrderAsync(int customerId, string customerName)
    {
        // Use like any other method -- zero allocation, pre-compiled
        var orderId = await _repository.CreateAsync(customerId);
        LogOrderCreated(orderId, customerName);
    }
}
```

> [!ad-note]
> The class must be declared `partial` for source generators to work. The generated code handles the `IsEnabled()` check, avoids boxing value types, and pre-parses the message template -- all at compile time.

## When to Use High-Performance Logging

| Scenario | Regular Logging | High-Performance Logging |
|---|---|---|
| Controller actions | Yes | Overkill |
| Service-layer business logic | Yes | Usually unnecessary |
| Inner loops processing thousands of items | No | Yes |
| Middleware on every request | Yes for most | Yes if the app handles 10K+ RPS |
| Library code shared across many applications | No | Yes |

> [!summary] Section Summary
> - `LoggerMessage.Define` and `[LoggerMessage]` source generators provide zero-allocation, pre-compiled logging
> - The source generator approach (`.NET 8+`) is cleaner and handles the `IsEnabled()` check automatically
> - Use high-performance logging in hot paths (inner loops, high-RPS middleware, library code)
> - For normal application code (controllers, services), standard `ILogger` methods are fine
