---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---


.NET 8 introduced the **`IExceptionHandler`** interface as the modern, DI-friendly way to handle exceptions. It replaces the need for custom exception middleware in many scenarios and supports multiple handlers in priority order.

```csharp
public interface IExceptionHandler
{
    ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken);
}
```

The handler returns `true` if it handled the exception (stopping the chain) or `false` to pass it to the next handler.

## Implementing IExceptionHandler

```csharp
// Handler for domain-specific exceptions
public class DomainExceptionHandler : IExceptionHandler
{
    private readonly ILogger<DomainExceptionHandler> _logger;

    public DomainExceptionHandler(ILogger<DomainExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        if (exception is not DomainException domainException)
            return false;  // Not our exception type -- pass to next handler

        _logger.LogWarning(exception,
            "Domain exception: {ErrorCode} on {Path}",
            domainException.ErrorCode,
            httpContext.Request.Path);

        httpContext.Response.StatusCode = domainException.StatusCode;

        await httpContext.Response.WriteAsJsonAsync(new ProblemDetails
        {
            Type = $"https://httpstatuses.com/{domainException.StatusCode}",
            Title = ReasonPhrases.GetReasonPhrase(domainException.StatusCode),
            Status = domainException.StatusCode,
            Detail = domainException.Message,
            Instance = httpContext.Request.Path,
            Extensions =
            {
                ["errorCode"] = domainException.ErrorCode,
                ["traceId"] = httpContext.TraceIdentifier
            }
        }, cancellationToken);

        return true;  // Handled -- stop the chain
    }
}

// Catch-all handler for unexpected exceptions
public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;
    private readonly IHostEnvironment _environment;

    public GlobalExceptionHandler(
        ILogger<GlobalExceptionHandler> logger,
        IHostEnvironment environment)
    {
        _logger = logger;
        _environment = environment;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception,
            "Unhandled exception on {Method} {Path}",
            httpContext.Request.Method,
            httpContext.Request.Path);

        httpContext.Response.StatusCode = 500;

        await httpContext.Response.WriteAsJsonAsync(new ProblemDetails
        {
            Type = "https://httpstatuses.com/500",
            Title = "Internal Server Error",
            Status = 500,
            Detail = _environment.IsDevelopment()
                ? exception.Message
                : "An unexpected error occurred.",
            Instance = httpContext.Request.Path,
            Extensions =
            {
                ["traceId"] = httpContext.TraceIdentifier
            }
        }, cancellationToken);

        return true;
    }
}
```

## Registration -- Order Matters

Handlers are tried **in registration order**. The first handler that returns `true` stops the chain.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register handlers in priority order -- most specific first
builder.Services.AddExceptionHandler<DomainExceptionHandler>();
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();

// Still need to add the exception handler middleware
builder.Services.AddProblemDetails();

var app = builder.Build();

// UseExceptionHandler() now uses the registered IExceptionHandler implementations
app.UseExceptionHandler();

app.MapControllers();
app.Run();
```

## Advantages Over Custom Middleware

| Aspect | Custom Middleware | IExceptionHandler (.NET 8+) |
|---|---|---|
| **DI support** | Constructor injection only (singleton lifetime) | Full DI with scoped services |
| **Multiple handlers** | Single middleware class handles all cases | Chain of handlers, each with single responsibility |
| **Testability** | Must test through the middleware pipeline | Can unit test each handler independently |
| **Registration** | `app.UseMiddleware<>()` | `builder.Services.AddExceptionHandler<>()` |
| **Integration** | Standalone | Works with built-in `UseExceptionHandler` and [[Problem Details]] service |

> [!ad-note]
> Even with `IExceptionHandler`, you still need to call `app.UseExceptionHandler()` to install the exception handling middleware. The `IExceptionHandler` implementations are called *by* that middleware when it catches an exception. Without `UseExceptionHandler()`, the handlers are never invoked.

> [!summary] Section Summary
> - `IExceptionHandler` (.NET 8+) is the modern approach to exception handling with full DI support
> - Multiple handlers are registered and tried in order -- the first to return `true` stops the chain
> - Most specific handlers (domain exceptions) should be registered before the catch-all handler
> - Each handler has a single responsibility and can be unit tested independently
> - You still need `app.UseExceptionHandler()` in the pipeline -- `IExceptionHandler` implementations run within that middleware
