---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
  - request
---

## Short-Circuiting

**Short-circuiting** occurs when a middleware component decides NOT to call `next()`, instead generating a response directly. This stops the request from flowing further down the pipeline.

### How Static File Short-Circuiting Works

The most common example of short-circuiting is the **static file middleware**:

```
Request: GET /css/site.css

[ExceptionHandler] --> [HSTS] --> [HTTPS Redirect] --> [StaticFiles]
                                                            |
                                                      File found!
                                                      Returns CSS
                                                      Does NOT call next()
                                                            |
                                                      <-- Response flows back

[Routing] -- NEVER REACHED
[Authentication] -- NEVER REACHED
[Authorization] -- NEVER REACHED
[MapControllers] -- NEVER REACHED
```

The static file middleware checks if the requested path matches a file in `wwwroot`. If it finds `wwwroot/css/site.css`, it reads the file, sets appropriate content-type headers, writes the file to the response, and returns -- without ever calling `next()`.

### Custom Short-Circuiting Example

You can short-circuit in your own middleware. Here is a **maintenance mode** middleware that returns a 503 for all requests when maintenance is active:

```csharp
public class MaintenanceModeMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IMaintenanceService _maintenanceService;

    public MaintenanceModeMiddleware(RequestDelegate next, IMaintenanceService maintenanceService)
    {
        _next = next;
        _maintenanceService = maintenanceService;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        if (_maintenanceService.IsMaintenanceActive)
        {
            // Short-circuit: do NOT call _next
            context.Response.StatusCode = StatusCodes.Status503ServiceUnavailable;
            context.Response.ContentType = "application/json";
            await context.Response.WriteAsJsonAsync(new
            {
                Error = "Service temporarily unavailable for maintenance",
                EstimatedReturn = _maintenanceService.EstimatedReturnTime
            });
            return; // Pipeline stops here
        }

        await _next(context); // Normal flow continues
    }
}
```

### Another Example: API Key Validation Short-Circuit

```csharp
app.Use(async (context, next) =>
{
    if (context.Request.Path.StartsWithSegments("/api"))
    {
        if (!context.Request.Headers.TryGetValue("X-Api-Key", out var apiKey)
            || apiKey != "expected-key-value")
        {
            context.Response.StatusCode = StatusCodes.Status401Unauthorized;
            await context.Response.WriteAsJsonAsync(new { Error = "Invalid API key" });
            return; // Short-circuit -- no further middleware runs
        }
    }

    await next(context); // Non-API requests pass through
});
```

> [!warning] Common Misconception
> Short-circuiting does not skip the "response phase" of middleware that already ran. If ExceptionHandler, HSTS, and HTTPS Redirect already processed the request before StaticFiles short-circuits, the response still flows back through those three middleware components on the way out. Only middleware AFTER the short-circuiting point is skipped entirely.

> [!summary] Section Summary
> Short-circuiting occurs when middleware returns a response without calling `next()`. Static file middleware is the canonical example -- when it finds a matching file, routing, authentication, and authorization never execute. Custom middleware can short-circuit for scenarios like maintenance mode, API key validation, or IP blocking.
