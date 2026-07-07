---
tags:
  - csharp
  - asp-net-core
  - filters
  - controllers
  - cross-cutting
---


There are three main approaches to applying filters, each with different trade-offs around dependency injection support.

### As Attributes (Simple)

Inherit from a base attribute class that implements the filter interface. These classes are both attributes and filters, so they can be applied directly with `[SquareBracket]` syntax.

```csharp
public class SimpleLogFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        // No access to DI services here -- cannot inject via constructor
        Debug.WriteLine($"Executing: {context.ActionDescriptor.DisplayName}");
    }

    public override void OnActionExecuted(ActionExecutedContext context)
    {
        Debug.WriteLine($"Executed: {context.ActionDescriptor.DisplayName}");
    }
}

// Usage
[SimpleLogFilter]
public class ProductsController : Controller
{
    [SimpleLogFilter] // Can also apply at action level
    public IActionResult GetAll() => Ok();
}
```

Available base attribute classes:
- `ActionFilterAttribute` -- implements both `IActionFilter` and `IResultFilter`
- `ExceptionFilterAttribute` -- implements `IExceptionFilter`
- `ResultFilterAttribute` -- implements `IResultFilter`

```ad-warning
The attribute approach **cannot use constructor dependency injection**. Attribute constructors only accept compile-time constants. If your filter needs injected services (loggers, database contexts, configuration), use `[TypeFilter]` or `[ServiceFilter]` instead.
```

### TypeFilter -- DI-Resolved via Attribute

`[TypeFilter]` tells the framework to create the filter using the DI container, resolving constructor dependencies automatically.

```csharp
public class LogActionFilter : IActionFilter
{
    private readonly ILogger<LogActionFilter> _logger;
    private readonly string _source;

    // ILogger injected from DI, source passed via Arguments
    public LogActionFilter(ILogger<LogActionFilter> logger, string source)
    {
        _logger = logger;
        _source = source;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        _logger.LogInformation("[{Source}] Executing: {Action}",
            _source, context.ActionDescriptor.DisplayName);
    }

    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Usage -- DI resolves ILogger, "Catalog" passed as extra argument
[TypeFilter(typeof(LogActionFilter), Arguments = new object[] { "Catalog" })]
public class CatalogController : Controller
{
    // ...
}
```

```ad-note
`TypeFilter` does **not** require the filter to be registered in the DI container. It creates the instance on the fly, resolving constructor parameters from the container where possible and from `Arguments` for the rest.
```

### ServiceFilter -- DI-Resolved from Services

`[ServiceFilter]` resolves the filter directly from the DI container. The filter **must** be explicitly registered as a service.

```csharp
// Register in Program.cs
builder.Services.AddScoped<PerformanceTimingFilter>();

// The filter class (same as any DI-resolved filter)
public class PerformanceTimingFilter : IActionFilter
{
    private readonly ILogger<PerformanceTimingFilter> _logger;

    public PerformanceTimingFilter(ILogger<PerformanceTimingFilter> logger)
    {
        _logger = logger;
    }

    // ...filter methods...
}

// Usage
[ServiceFilter(typeof(PerformanceTimingFilter))]
public class OrdersController : Controller
{
    // ...
}
```

```ad-tip
`ServiceFilter` is more explicit than `TypeFilter` -- the filter must be registered, so you get a clear error at startup if the registration is missing. It also gives you control over the service lifetime (transient, scoped, singleton).
```

### Comparison Table

| Approach | DI Support | Registration Required | Extra Arguments | Best For |
|---|---|---|---|---|
| Attribute | No | No | No | Simple filters with no dependencies |
| TypeFilter | Yes | No | Yes (`Arguments`) | Filters needing DI + extra config |
| ServiceFilter | Yes | Yes (explicit) | No | Filters already registered in DI |
