---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
---

## The Three Fundamental Methods

ASP.NET Core provides three fundamental methods for adding middleware to the pipeline: **`app.Use()`**, **`app.Run()`**, and **`app.Map()`**. Understanding the differences between them is essential.

### app.Use() -- Chainable Middleware

**`app.Use()`** adds a middleware that **can call `next()`** to pass the request to the next middleware. It is the most common method and is used for middleware that needs to do work both before and after subsequent middleware.

```csharp
// app.Use() -- always call next() unless you want to short-circuit
app.Use(async (context, next) =>
{
    // Check for an API key on all requests
    if (!context.Request.Headers.ContainsKey("X-Api-Key"))
    {
        context.Response.StatusCode = 401;
        await context.Response.WriteAsync("API key is required");
        return;  // Short-circuit: do NOT call next()
    }

    // API key present -- continue to next middleware
    await next();
});
```

### app.Run() -- Terminal Middleware

**`app.Run()`** adds a **terminal middleware** -- it does **not** receive a `next` parameter and therefore cannot pass the request further down the pipeline. It always ends the pipeline.

```csharp
// app.Run() -- terminal, no next() available
app.Run(async context =>
{
    // This is the end of the pipeline
    await context.Response.WriteAsync("Request handled by terminal middleware");
});

// WARNING: Any middleware registered after app.Run() will NEVER execute
app.Use(async (context, next) =>
{
    // This code is unreachable!
    Console.WriteLine("This will never print");
    await next();
});
```

> [!danger]
> Never place `app.Run()` before other middleware unless you intentionally want to terminate the pipeline at that point. Any middleware added after `app.Run()` is dead code and will never execute.

### app.Map() -- Branch the Pipeline

**`app.Map()`** creates a **branch** in the pipeline based on the request path. When the request path matches the specified prefix, the request is routed to a separate middleware pipeline. The matched path segment is removed from `HttpContext.Request.Path` and appended to `HttpContext.Request.PathBase`.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Main pipeline middleware
app.Use(async (context, next) =>
{
    Console.WriteLine("Main pipeline: before branch");
    await next();
    Console.WriteLine("Main pipeline: after branch");
});

// Branch for /api requests
app.Map("/api", apiApp =>
{
    apiApp.Use(async (context, next) =>
    {
        Console.WriteLine($"API branch: handling {context.Request.Path}");
        await next();
    });

    apiApp.Run(async context =>
    {
        await context.Response.WriteAsync("API response");
    });
});

// Branch for /health requests
app.Map("/health", healthApp =>
{
    healthApp.Run(async context =>
    {
        await context.Response.WriteAsync("Healthy");
    });
});

// Fallback for unmatched routes
app.Run(async context =>
{
    await context.Response.WriteAsync("Default response");
});

app.Run();
```

There is also **`app.MapWhen()`** which branches based on any predicate, not just a path:

```csharp
// Branch based on a custom condition (not just path)
app.MapWhen(
    context => context.Request.Headers.ContainsKey("X-Custom-Header"),
    customApp =>
    {
        customApp.Run(async context =>
        {
            await context.Response.WriteAsync("Custom header detected -- special handling");
        });
    });
```

### Comparison Table

| Method | Receives `next`? | Terminal? | Use Case |
|---|---|---|---|
| `app.Use()` | Yes | No (unless you skip `next()`) | Most middleware: logging, auth checks, header manipulation |
| `app.Run()` | No | Always | Final handler, fallback responses, simple endpoints |
| `app.Map()` | N/A (creates branch) | Creates sub-pipeline | Path-based routing to separate pipelines |
| `app.MapWhen()` | N/A (creates branch) | Creates sub-pipeline | Condition-based branching (any predicate) |
| `app.UseWhen()` | N/A (conditional) | Rejoins main pipeline | Conditionally add middleware but stay in main pipeline |

> [!tip]
> Use `app.UseWhen()` instead of `app.MapWhen()` when you want to conditionally apply middleware but still continue in the main pipeline afterward. `MapWhen` creates a true fork -- the request never returns to the main pipeline. `UseWhen` runs the middleware and then rejoins.

> [!summary] Section Summary
> `app.Use()` is chainable middleware that calls `next()` to continue. `app.Run()` is terminal middleware that ends the pipeline. `app.Map()` branches the pipeline based on path. Choose `Use` for most middleware, `Run` for terminal handlers, and `Map`/`MapWhen`/`UseWhen` for conditional branching.
