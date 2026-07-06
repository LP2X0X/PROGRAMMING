---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---


Every `ILogger` instance has a **category** -- a string that identifies the source of the log entry. When you inject `ILogger<ProductsController>`, the category is the full type name: `"MyApp.Controllers.ProductsController"`.

Categories serve two purposes:
1. **Identification** -- you can see which class generated each log entry
2. **Filtering** -- you can configure different log levels per category

```csharp
// Category: "MyApp.Controllers.ProductsController"
public class ProductsController : Controller
{
    private readonly ILogger<ProductsController> _logger;
    // ...
}

// Category: "MyApp.Services.OrderService"
public class OrderService
{
    private readonly ILogger<OrderService> _logger;
    // ...
}

// Custom category name (rare, but useful for cross-cutting concerns)
public class CacheService
{
    private readonly ILogger _logger;

    public CacheService(ILoggerFactory factory)
    {
        // Category: "Caching"
        _logger = factory.CreateLogger("Caching");
    }
}
```

> [!ad-note]
> The category string uses the namespace hierarchy, which is why per-category filtering works with prefixes. Setting `"MyApp.Services": "Warning"` in configuration affects all loggers in the `MyApp.Services` namespace.

> [!summary] Section Summary
> - Log categories identify the source of each log entry and enable per-source filtering
> - `ILogger<T>` automatically uses the full type name of `T` as the category
> - Use `ILoggerFactory.CreateLogger("name")` for custom category names
> - Categories follow the namespace hierarchy, enabling prefix-based filtering
