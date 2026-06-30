---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
---

## Inline Middleware with app.Use

The simplest way to write middleware is **inline** using `app.Use()` with a lambda. This is excellent for quick, focused middleware. Here is a complete working example of a real-world scenario -- an order validation middleware:

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Http;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Middleware 1: Request correlation ID
app.Use(async (context, next) =>
{
    // Check if the client sent a correlation ID; generate one if not
    if (!context.Request.Headers.TryGetValue("X-Correlation-Id", out var correlationId))
    {
        correlationId = Guid.NewGuid().ToString();
    }

    // Store it for downstream middleware and services
    context.Items["CorrelationId"] = correlationId.ToString();

    // Add it to the response so the client can trace it
    context.Response.OnStarting(() =>
    {
        context.Response.Headers["X-Correlation-Id"] = correlationId.ToString();
        return Task.CompletedTask;
    });

    await next();
});

// Middleware 2: Request/Response logging
app.Use(async (context, next) =>
{
    var correlationId = context.Items["CorrelationId"] as string;
    var method = context.Request.Method;
    var path = context.Request.Path;

    Console.WriteLine($"[{correlationId}] --> {method} {path}");

    await next();

    Console.WriteLine($"[{correlationId}] <-- {context.Response.StatusCode}");
});

// Middleware 3: Simple API key validation
app.Use(async (context, next) =>
{
    // Skip auth for health check endpoint
    if (context.Request.Path.StartsWithSegments("/health"))
    {
        await next();
        return;
    }

    var apiKey = context.Request.Headers["X-Api-Key"].FirstOrDefault();

    if (string.IsNullOrEmpty(apiKey) || apiKey != "my-secret-key-12345")
    {
        context.Response.StatusCode = StatusCodes.Status401Unauthorized;
        await context.Response.WriteAsJsonAsync(new
        {
            error = "Invalid or missing API key",
            correlationId = context.Items["CorrelationId"]
        });
        return;  // Short-circuit -- do NOT call next()
    }

    await next();
});

// Terminal middleware -- handle requests
app.Map("/api/orders", orderApp =>
{
    orderApp.Run(async context =>
    {
        await context.Response.WriteAsJsonAsync(new
        {
            orders = new[]
            {
                new { id = 1, product = "Widget", quantity = 5 },
                new { id = 2, product = "Gadget", quantity = 3 }
            }
        });
    });
});

app.Map("/health", healthApp =>
{
    healthApp.Run(async context =>
    {
        await context.Response.WriteAsync("OK");
    });
});

app.Run(async context =>
{
    context.Response.StatusCode = 404;
    await context.Response.WriteAsJsonAsync(new { error = "Not found" });
});

app.Run();
```

> [!ad-note]
> Inline middleware is great for small, focused concerns. For more complex middleware with dependencies (injected services, configuration), prefer writing a [[Custom Middleware]] class with the convention-based or factory-based approach.

> [!summary] Section Summary
> Inline middleware uses `app.Use(async (context, next) => { ... })` for quick, lambda-based middleware. It is ideal for simple cross-cutting concerns like correlation IDs, logging, and basic validation. For complex middleware with dependencies, extract it into a dedicated class.
