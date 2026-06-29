---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---


ASP.NET Core provides two mechanisms for handling exceptions: **exception filters** (part of MVC) and **exception middleware** (part of the pipeline). They serve different purposes and have different scopes.

## Exception Filters

Exception filters implement `IExceptionFilter` or `IAsyncExceptionFilter` and are part of the **MVC filter pipeline**. They only catch exceptions that occur within MVC action execution.

```csharp
public class CustomExceptionFilter : IExceptionFilter
{
    private readonly ILogger<CustomExceptionFilter> _logger;

    public CustomExceptionFilter(ILogger<CustomExceptionFilter> logger)
    {
        _logger = logger;
    }

    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception,
            "Exception in {Controller}.{Action}",
            context.RouteData.Values["controller"],
            context.RouteData.Values["action"]);

        context.Result = new ObjectResult(new
        {
            error = "An error occurred",
            traceId = context.HttpContext.TraceIdentifier
        })
        {
            StatusCode = 500
        };

        // Mark the exception as handled -- stops propagation
        context.ExceptionHandled = true;
    }
}

// Registration -- globally or per controller/action
builder.Services.AddControllers(options =>
{
    options.Filters.Add<CustomExceptionFilter>();
});
```

## Comparison Table

| Aspect | Exception Filters | Exception Middleware |
|---|---|---|
| **Scope** | MVC actions and Razor Pages only | Entire request pipeline |
| **Catches from** | Controller actions, action filters, result filters | Any middleware, controllers, Razor Pages, minimal APIs |
| **Registration** | `options.Filters.Add<>()` or `[TypeFilter]` attribute | `app.UseMiddleware<>()` or `app.UseExceptionHandler()` |
| **DI support** | Yes, via `[TypeFilter]` or `[ServiceFilter]` | Yes, via constructor injection |
| **Access to MVC context** | Yes -- `ExceptionContext` with `ActionDescriptor`, `RouteData`, etc. | No -- only `HttpContext` |
| **Can set `IActionResult`** | Yes -- `context.Result = new ObjectResult(...)` | No -- must write directly to `HttpResponse` |
| **Order of execution** | Runs within the MVC pipeline (after model binding, before result execution) | Runs at the middleware level (outermost layer) |

> [!tip]
> Use exception filters when you need MVC-specific context (which controller, which action, route data) and the exception originates from controller logic. Use exception middleware for everything else -- including exceptions from other middleware, minimal API endpoints, and as a global safety net. In most applications, **use both**: middleware as the outermost catch-all, and filters for MVC-specific error formatting.

> [!summary] Section Summary
> - Exception filters are MVC-specific and catch exceptions from controller actions, with access to `ActionDescriptor`, `RouteData`, and the ability to set `IActionResult`
> - Exception middleware operates at the pipeline level and catches exceptions from all sources
> - Filters cannot catch exceptions from middleware, minimal APIs, or non-MVC code
> - Best practice: use middleware as the global safety net and filters for MVC-specific formatting
