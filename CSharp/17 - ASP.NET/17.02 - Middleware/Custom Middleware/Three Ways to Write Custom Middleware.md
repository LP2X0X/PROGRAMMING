---
tags:
  - csharp
  - asp-net-core
  - middleware
  - custom
---

## Three Ways to Write Custom Middleware

ASP.NET Core provides three distinct approaches for writing custom middleware. Each has different trade-offs around lifetime management, dependency injection support, and complexity.

### 1. Inline Middleware with app.Use()

The simplest approach is **inline middleware** -- a lambda passed directly to `app.Use()` in `Program.cs`. The lambda receives the `HttpContext` and a `RequestDelegate` representing the next middleware in the pipeline.

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();

var app = builder.Build();

// Inline middleware: log request method and path
app.Use(async (HttpContext context, RequestDelegate next) =>
{
    var method = context.Request.Method;
    var path = context.Request.Path;
    
    Console.WriteLine($"[Request] {method} {path} started at {DateTime.UtcNow}");
    
    await next(context); // Call the next middleware
    
    var statusCode = context.Response.StatusCode;
    Console.WriteLine($"[Response] {method} {path} completed with {statusCode}");
});

app.MapControllers();
app.Run();
```

> [!ad-note]
> The `next(context)` call is what passes control to the next middleware. If you omit it, the pipeline is **short-circuited** and no downstream middleware (including your controllers) will execute. This is intentional for scenarios like authentication rejection.

**When to use inline middleware:**
- Quick prototyping or debugging
- Very simple logic (a few lines)
- One-off concerns that don't warrant a full class

> [!warning] Common Misconception
> Some developers assume `app.Use()` and `app.Run()` are interchangeable. They are not. `app.Run()` is a **terminal middleware** -- it does not receive a `next` delegate and always ends the pipeline. Use `app.Use()` when you need to pass the request to the next middleware.

> [!summary] Section Summary
> Inline middleware with `app.Use()` is the quickest way to add custom logic to the pipeline. Pass a lambda that takes `HttpContext` and `RequestDelegate`, call `next(context)` to continue the pipeline, and optionally inspect the response afterward. Best for simple, throwaway, or prototype scenarios.

### 2. Convention-Based Middleware Class

**Convention-based middleware** is the most common approach in production applications. The class follows a naming convention rather than implementing an interface -- ASP.NET Core discovers the correct method by name.

The conventions are:
1. The constructor must accept a `RequestDelegate` parameter (the next middleware)
2. The class must have a public method named `Invoke` or `InvokeAsync` that accepts `HttpContext` as its first parameter and returns `Task`
3. The class is instantiated **once** at application startup (singleton lifetime)

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    // Constructor: receives the next middleware delegate
    // and any SINGLETON services via constructor injection
    public RequestLoggingMiddleware(
        RequestDelegate next,
        ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    // InvokeAsync is called for every HTTP request
    // Additional parameters beyond HttpContext are resolved from DI per-request
    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation(
            "Handling {Method} {Path}",
            context.Request.Method,
            context.Request.Path);

        await _next(context);

        _logger.LogInformation(
            "Finished {Method} {Path} with status {StatusCode}",
            context.Request.Method,
            context.Request.Path,
            context.Response.StatusCode);
    }
}
```

Register it in the pipeline:

```csharp
var app = builder.Build();

app.UseMiddleware<RequestLoggingMiddleware>();

app.MapControllers();
app.Run();
```

> [!info] Key Detail
> Convention-based middleware has **singleton lifetime** by default. The constructor runs once, and the same instance handles every request for the lifetime of the application. This is critical for understanding dependency injection behavior -- see [[#The Scoped Service Trap]].

> [!summary] Section Summary
> Convention-based middleware uses a class with a constructor accepting `RequestDelegate` and an `InvokeAsync(HttpContext)` method. It is created once (singleton) at startup. This is the standard approach for production middleware and supports constructor injection for singleton services and method injection for scoped/transient services.

### 3. Factory-Based Middleware with IMiddleware

**Factory-based middleware** implements the `IMiddleware` interface explicitly. Unlike convention-based middleware, the DI container creates a new instance per request (or per the registered lifetime).

```csharp
public class TenantResolutionMiddleware : IMiddleware
{
    private readonly ITenantService _tenantService;

    // Constructor injection works with ANY service lifetime
    // because the middleware itself is resolved from DI per-request
    public TenantResolutionMiddleware(ITenantService tenantService)
    {
        _tenantService = tenantService;
    }

    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        var tenantId = context.Request.Headers["X-Tenant-Id"].FirstOrDefault();
        
        if (!string.IsNullOrEmpty(tenantId))
        {
            var tenant = await _tenantService.ResolveTenantAsync(tenantId);
            context.Items["Tenant"] = tenant;
        }

        await next(context);
    }
}
```

> [!warning] Common Misconception
> Many developers forget that `IMiddleware` implementations **must be registered in the DI container**. Without this registration, you will get a runtime exception. Convention-based middleware does not require explicit DI registration.

Registration requires two steps:

```csharp
// Step 1: Register the middleware class in the DI container
builder.Services.AddTransient<TenantResolutionMiddleware>();

// Step 2: Add it to the pipeline
var app = builder.Build();
app.UseMiddleware<TenantResolutionMiddleware>();
app.MapControllers();
app.Run();
```

> [!ad-note]
> The lifetime you register (`AddTransient`, `AddScoped`, `AddSingleton`) determines how often a new middleware instance is created. `AddScoped` gives you one instance per request, which is the most common choice for `IMiddleware`.

> [!summary] Section Summary
> Factory-based middleware implements `IMiddleware` and is resolved from the DI container per request. This means you can safely inject scoped services into the constructor. The trade-off is that you must explicitly register the middleware class in DI, and a new instance is created per request (unless registered as singleton).

### Comparison of Approaches

| Approach | Lifetime | DI Support | Registration | Best For |
|---|---|---|---|---|
| Inline `app.Use()` | N/A (closure) | Captures from closure only | Inline in `Program.cs` | Quick prototyping, simple one-off logic |
| Convention-based class | Singleton (one instance) | Constructor: singleton only. `InvokeAsync` params: any lifetime | `app.UseMiddleware<T>()` | Most production middleware, standard approach |
| Factory-based `IMiddleware` | Controlled by DI registration | Full DI support in constructor (any lifetime) | DI registration + `app.UseMiddleware<T>()` | Middleware that needs scoped dependencies in constructor |

> [!summary] Section Summary
> Choose inline for prototypes, convention-based for most production middleware, and factory-based `IMiddleware` when your middleware depends on scoped services and you want clean constructor injection. Convention-based is the default choice in the ASP.NET Core ecosystem.
