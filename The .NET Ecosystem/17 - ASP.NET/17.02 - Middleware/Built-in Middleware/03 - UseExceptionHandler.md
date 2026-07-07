---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseExceptionHandler

**`UseExceptionHandler`** is the production-grade exception handling middleware. It catches any unhandled exception thrown by downstream middleware and generates an appropriate error response without exposing sensitive details to the client.

### How It Works

When an exception propagates up the pipeline, this middleware:
1. Catches the exception
2. Logs it
3. Clears the response
4. Re-executes the pipeline using a specified error-handling path (e.g., `/Error`)
5. The error-handling endpoint can inspect `IExceptionHandlerPathFeature` to access the original exception

### Configuration for MVC (HTML Error Page)

```csharp
// Program.cs
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
}
```

```csharp
// HomeController.cs
[AllowAnonymous]
[ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
public IActionResult Error()
{
    var exceptionFeature = HttpContext.Features.Get<IExceptionHandlerPathFeature>();

    // Log the exception
    _logger.LogError(exceptionFeature?.Error, 
        "Unhandled exception at path: {Path}", 
        exceptionFeature?.Path);

    return View(new ErrorViewModel
    {
        RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier,
        Message = "An unexpected error occurred. Please try again later."
    });
}
```

### Configuration for Web API (JSON Error Response)

```csharp
// Program.cs
app.UseExceptionHandler(appError =>
{
    appError.Run(async context =>
    {
        context.Response.StatusCode = StatusCodes.Status500InternalServerError;
        context.Response.ContentType = "application/json";

        var exceptionFeature = context.Features.Get<IExceptionHandlerPathFeature>();
        var exception = exceptionFeature?.Error;

        var logger = context.RequestServices.GetRequiredService<ILogger<Program>>();
        logger.LogError(exception, "Unhandled exception in request pipeline");

        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status500InternalServerError,
            Title = "Internal Server Error",
            Detail = app.Environment.IsDevelopment() 
                ? exception?.Message 
                : "An error occurred processing your request."
        };

        await context.Response.WriteAsJsonAsync(problemDetails);
    });
});
```

### .NET 8+ `IExceptionHandler` Interface

Starting with .NET 8, you can register custom exception handlers via dependency injection:

```csharp
// OrderExceptionHandler.cs
public class OrderExceptionHandler : IExceptionHandler
{
    private readonly ILogger<OrderExceptionHandler> _logger;

    public OrderExceptionHandler(ILogger<OrderExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception, "Exception while processing order request");

        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status500InternalServerError,
            Title = "Server Error",
            Type = "https://tools.ietf.org/html/rfc7231#section-6.6.1"
        };

        httpContext.Response.StatusCode = StatusCodes.Status500InternalServerError;
        await httpContext.Response.WriteAsJsonAsync(problemDetails, cancellationToken);

        return true; // true = exception was handled
    }
}

// Program.cs
builder.Services.AddExceptionHandler<OrderExceptionHandler>();
app.UseExceptionHandler();
```

### When You Need It

Always in production. Every ASP.NET Core application should have `UseExceptionHandler` registered for non-development environments.

### Gotchas

- Must be placed **very early** in the pipeline so it catches exceptions from all downstream middleware
- The re-execution path (`/Error`) runs through the entire pipeline again, so ensure the error endpoint cannot itself throw
- Response headers that were already sent before the exception cannot be modified -- the middleware clears the response only if headers have not yet been flushed

> [!summary] Section Summary
> `UseExceptionHandler` is the backbone of production error handling. For MVC apps, it redirects to an error page. For APIs, it returns structured JSON (ideally `ProblemDetails`). In .NET 8+, the `IExceptionHandler` interface offers a cleaner DI-friendly approach. Always place it at the outermost layer of the pipeline.
