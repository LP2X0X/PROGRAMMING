---
tags:
  - csharp
  - asp-net-core
  - middleware
  - custom
---

## Extension Method Pattern

Production middleware is typically exposed through an **extension method** on `IApplicationBuilder`. This provides a clean, discoverable API: `app.UseRequestTiming()` instead of `app.UseMiddleware<RequestTimingMiddleware>()`.

```csharp
// The middleware class
public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;

    public RequestTimingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();
        
        context.Response.OnStarting(() =>
        {
            stopwatch.Stop();
            context.Response.Headers["X-Response-Time"] = 
                $"{stopwatch.ElapsedMilliseconds}ms";
            return Task.CompletedTask;
        });

        await _next(context);
    }
}

// The extension method (typically in a separate static class)
public static class RequestTimingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestTiming(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestTimingMiddleware>();
    }
}
```

Usage becomes clean and follows the same pattern as built-in middleware:

```csharp
var app = builder.Build();

app.UseRequestTiming();   // Custom -- looks just like built-in
app.UseAuthentication();  // Built-in
app.UseAuthorization();   // Built-in

app.MapControllers();
app.Run();
```

**With configuration options:**

```csharp
public class RequestTimingOptions
{
    public string HeaderName { get; set; } = "X-Response-Time";
    public bool IncludeMilliseconds { get; set; } = true;
}

public static class RequestTimingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestTiming(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestTimingMiddleware>();
    }

    public static IApplicationBuilder UseRequestTiming(
        this IApplicationBuilder builder,
        Action<RequestTimingOptions> configureOptions)
    {
        var options = new RequestTimingOptions();
        configureOptions(options);
        return builder.UseMiddleware<RequestTimingMiddleware>(Options.Create(options));
    }
}
```

> [!ad-note]
> The `UseMiddleware<T>()` method accepts additional constructor arguments after `RequestDelegate`. You can pass `IOptions<T>` wrappers or other values directly. However, the preferred approach for complex configuration is to use the Options pattern with `builder.Services.Configure<T>()`.

> [!summary] Section Summary
> Wrap middleware registration in an `IApplicationBuilder` extension method for clean, discoverable APIs. Follow the `UseXxx()` naming convention. Support optional configuration through an `Action<TOptions>` overload or the Options pattern.
