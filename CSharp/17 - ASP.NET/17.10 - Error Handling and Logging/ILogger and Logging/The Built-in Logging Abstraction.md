---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---

> [!abstract] Overview
> Logging is the primary way you observe what your application is doing in production. When something goes wrong at 3 AM, you are not stepping through code with a debugger -- you are reading logs. The quality of your logging directly determines how quickly you can diagnose issues, understand user behavior, and verify that your application is functioning correctly.
>
> ASP.NET Core provides a **built-in logging abstraction** (`ILogger<T>`) that decouples your application code from any specific logging framework. You write log statements against the abstraction, and the actual destination (console, file, Elasticsearch, Application Insights) is determined by configuration -- not by changing your code.


ASP.NET Core includes a logging system in the `Microsoft.Extensions.Logging` namespace. The central interface is **`ILogger<T>`**, which you inject via [[DI Overview|dependency injection]] everywhere you need to log.

```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;

    public OrderService(ILogger<OrderService> logger)
    {
        _logger = logger;
    }

    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        _logger.LogInformation("Creating order for customer {CustomerId}", 
            request.CustomerId);

        var order = new Order { CustomerId = request.CustomerId };
        // ... business logic ...

        _logger.LogInformation("Order {OrderId} created successfully", 
            order.Id);

        return order;
    }
}
```

The key design decision behind `ILogger<T>` is the **abstraction layer**. Your code never references a specific logging library -- it only depends on `ILogger<T>`. This means you can switch from Console logging to Serilog to NLog without changing a single line of application code.

## The Logging Interfaces

| Interface | Purpose |
|---|---|
| `ILogger` | Base interface with `Log()`, `IsEnabled()`, `BeginScope()` |
| `ILogger<T>` | Generic version that sets the log **category** to the type name of `T` |
| `ILoggerFactory` | Creates `ILogger` instances; registered as a singleton in DI |
| `ILoggerProvider` | Represents a logging destination (Console, File, etc.) |

```csharp
// ILogger<T> is the most common -- inject it with the class type
public class ProductsController : Controller
{
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(ILogger<ProductsController> logger)
    {
        _logger = logger;
    }
}

// ILoggerFactory is useful when you need to create loggers dynamically
public class DynamicService
{
    private readonly ILoggerFactory _loggerFactory;

    public DynamicService(ILoggerFactory loggerFactory)
    {
        _loggerFactory = loggerFactory;
    }

    public void ProcessBatch(string batchName)
    {
        // Create a logger with a custom category name
        var logger = _loggerFactory.CreateLogger($"BatchProcessor.{batchName}");
        logger.LogInformation("Starting batch {BatchName}", batchName);
    }
}
```

> [!ad-note]
> `ILogger<T>` is registered in DI automatically when you call `WebApplication.CreateBuilder()`. You do not need to register it manually. The framework resolves it by creating an `ILogger` with the category set to the full name of `T`.

> [!summary] Section Summary
> - `ILogger<T>` is the primary logging interface, injected via DI throughout the application
> - It is an abstraction -- your code does not depend on any specific logging framework
> - `ILoggerFactory` creates loggers with custom category names for dynamic scenarios
> - The logging system is registered automatically by `WebApplication.CreateBuilder()`
> - This abstraction lets you switch logging providers (Console, Serilog, NLog) without changing application code
