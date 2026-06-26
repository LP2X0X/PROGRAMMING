---
tags:
  - csharp
  - asp-net-core
  - middleware
  - custom
aliases:
  - Custom ASP.NET Middleware
  - Writing Custom Middleware
  - Middleware Class
  - IMiddleware
status: complete
date: 2026-06-18
---

# Custom Middleware

**Custom middleware** is user-defined code that plugs into the ASP.NET Core [[Request Pipeline]] to inspect, modify, or short-circuit HTTP requests and responses. While ASP.NET Core ships with many [[Built-in Middleware]] components (authentication, static files, routing, etc.), real-world applications almost always need custom middleware for cross-cutting concerns like request logging, correlation IDs, global error handling, or API key validation.

This note covers the three ways to create custom middleware, how dependency injection interacts with middleware lifetime, real-world implementations, testing strategies, and when to choose middleware over action filters.

> [!ad-note]
> This note assumes familiarity with the [[Middleware Overview]] and [[Request Pipeline]] concepts. Custom middleware sits at the same level as built-in middleware -- the pipeline does not distinguish between them.

---

## Table of Contents

- [[#Three Ways to Write Custom Middleware]]
  - [[#1 Inline Middleware with app Use]]
  - [[#2 Convention-Based Middleware Class]]
  - [[#3 Factory-Based Middleware with IMiddleware]]
  - [[#Comparison of Approaches]]
- [[#Dependency Injection in Middleware]]
  - [[#Constructor Injection vs Method Injection]]
  - [[#The Scoped Service Trap]]
- [[#The Invoke and InvokeAsync Convention]]
- [[#Extension Method Pattern]]
- [[#Real-World Examples]]
  - [[#Request Timing Middleware]]
  - [[#Correlation ID Middleware]]
  - [[#API Key Authentication Middleware]]
  - [[#Global Exception Handling Middleware]]
- [[#Testing Custom Middleware]]
- [[#Middleware vs Action Filters]]
- [[#Related Topics]]
- [[#Further Reading]]
- [[#Comprehensive Summary]]

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

---

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

---

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

---

### Comparison of Approaches

| Approach | Lifetime | DI Support | Registration | Best For |
|---|---|---|---|---|
| Inline `app.Use()` | N/A (closure) | Captures from closure only | Inline in `Program.cs` | Quick prototyping, simple one-off logic |
| Convention-based class | Singleton (one instance) | Constructor: singleton only. `InvokeAsync` params: any lifetime | `app.UseMiddleware<T>()` | Most production middleware, standard approach |
| Factory-based `IMiddleware` | Controlled by DI registration | Full DI support in constructor (any lifetime) | DI registration + `app.UseMiddleware<T>()` | Middleware that needs scoped dependencies in constructor |

> [!summary] Section Summary
> Choose inline for prototypes, convention-based for most production middleware, and factory-based `IMiddleware` when your middleware depends on scoped services and you want clean constructor injection. Convention-based is the default choice in the ASP.NET Core ecosystem.

---

## Dependency Injection in Middleware

### Constructor Injection vs Method Injection

Convention-based middleware supports two forms of dependency injection:

**Constructor injection** -- resolved once at startup:

```csharp
public class OrderValidationMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<OrderValidationMiddleware> _logger;
    private readonly IConfiguration _configuration;

    // These services are resolved ONCE when the middleware is constructed
    public OrderValidationMiddleware(
        RequestDelegate next,
        ILogger<OrderValidationMiddleware> logger,
        IConfiguration configuration)
    {
        _next = next;
        _logger = logger;
        _configuration = configuration;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // _logger and _configuration are the same instances for every request
        await _next(context);
    }
}
```

**Method injection** -- resolved per request via `InvokeAsync` parameters:

```csharp
public class OrderValidationMiddleware
{
    private readonly RequestDelegate _next;

    public OrderValidationMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    // IOrderRepository is resolved from DI for EACH request
    public async Task InvokeAsync(
        HttpContext context,
        IOrderRepository orderRepository,
        IInventoryService inventoryService)
    {
        var orderId = context.Request.RouteValues["orderId"]?.ToString();
        
        if (orderId is not null)
        {
            var order = await orderRepository.GetByIdAsync(orderId);
            if (order is null)
            {
                context.Response.StatusCode = StatusCodes.Status404NotFound;
                return; // Short-circuit
            }
            context.Items["Order"] = order;
        }

        await next(context);
    }
}
```

> [!ad-note]
> ASP.NET Core's middleware activation logic inspects the `InvokeAsync` method signature. Any parameter beyond `HttpContext` is resolved from the DI container for each request. This is a convention -- there is no interface or attribute driving it.

> [!summary] Section Summary
> Constructor injection in convention-based middleware resolves services once at startup. Method injection via additional `InvokeAsync` parameters resolves services per request from the DI container. Use constructor injection for singleton services and method injection for scoped or transient services.

---

### The Scoped Service Trap

This is one of the most common bugs in ASP.NET Core middleware development.

Because convention-based middleware is **singleton** (constructed once), injecting a **scoped** service into the constructor captures a single instance that outlives its intended scope. The scoped service was designed to live for one request, but it now lives for the entire application lifetime.

```csharp
// BUG: InventoryDbContext is scoped, but middleware is singleton
public class InventoryCheckMiddleware
{
    private readonly RequestDelegate _next;
    private readonly InventoryDbContext _dbContext; // CAPTURED ONCE!

    public InventoryCheckMiddleware(
        RequestDelegate next,
        InventoryDbContext dbContext) // This is resolved at startup
    {
        _next = next;
        _dbContext = dbContext; // Same DbContext for ALL requests -- bug!
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // _dbContext is stale, not thread-safe, and may be disposed
        var count = await _dbContext.Products.CountAsync();
        await _next(context);
    }
}
```

> [!danger]
> This will cause `ObjectDisposedException`, stale data, and thread-safety issues. The `DbContext` was created for a scope that ended long ago, but the singleton middleware still holds a reference to it.

**The fix: inject scoped services into `InvokeAsync` instead:**

```csharp
public class InventoryCheckMiddleware
{
    private readonly RequestDelegate _next;

    public InventoryCheckMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    // InventoryDbContext is resolved fresh for each request
    public async Task InvokeAsync(
        HttpContext context,
        InventoryDbContext dbContext)
    {
        var count = await dbContext.Products.CountAsync();
        context.Response.Headers["X-Product-Count"] = count.ToString();
        await _next(context);
    }
}
```

**Why this works:** ASP.NET Core creates a new DI scope for each HTTP request. When `InventoryDbContext` is a parameter of `InvokeAsync`, it is resolved from the per-request scope, giving you a fresh instance each time.

> [!tip]
> If you enable `ValidateScopes` in development (which is on by default in `WebApplication.CreateBuilder`), ASP.NET Core will throw an `InvalidOperationException` at startup if you try to resolve a scoped service from the root provider. This catches the bug early -- but only in Development environment.

```csharp
// This is enabled by default in Development
builder.Host.UseDefaultServiceProvider(options =>
{
    options.ValidateScopes = true;
    options.ValidateOnBuild = true;
});
```

> [!summary] Section Summary
> Convention-based middleware is singleton, so injecting scoped services into the constructor captures a single instance that outlives its scope. Always inject scoped services (DbContext, repositories, unit-of-work) via `InvokeAsync` method parameters. Alternatively, use factory-based `IMiddleware` which supports scoped constructor injection natively. Enable `ValidateScopes` in development to catch this bug early.

---

## The Invoke and InvokeAsync Convention

Convention-based middleware relies on a **naming convention** for its entry point method. ASP.NET Core looks for a method named either `Invoke` or `InvokeAsync` on the middleware class.

The rules:
1. The method must be `public`
2. It must return `Task` (not `void`, not `Task<T>`)
3. The first parameter must be `HttpContext`
4. Additional parameters are resolved from DI per request
5. You must have exactly **one** such method -- not both `Invoke` and `InvokeAsync`

```csharp
// Preferred: async version
public async Task InvokeAsync(HttpContext context)
{
    // Pre-processing
    await _next(context);
    // Post-processing
}

// Also valid: synchronous-looking but still returns Task
public Task Invoke(HttpContext context)
{
    if (SomeCondition(context))
    {
        context.Response.StatusCode = 403;
        return Task.CompletedTask; // Short-circuit without calling next
    }

    return _next(context); // Pass-through
}
```

> [!warning] Common Misconception
> Some developers define both `Invoke` and `InvokeAsync` on the same class, expecting one to be used for sync and the other for async scenarios. ASP.NET Core will throw an `InvalidOperationException` if both are present. Pick one.

> [!tip]
> Prefer `InvokeAsync` over `Invoke`. The `Async` suffix communicates the asynchronous nature of HTTP processing, and most middleware will use `await` internally.

> [!summary] Section Summary
> The middleware entry point must be a single public method named `Invoke` or `InvokeAsync`, returning `Task`, with `HttpContext` as its first parameter. Prefer `InvokeAsync`. Additional parameters beyond `HttpContext` are resolved from the DI container per request.

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

---

## Real-World Examples

### Request Timing Middleware

Measures how long each request takes and adds the elapsed time as a response header. Useful for performance monitoring.

```csharp
using System.Diagnostics;

public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestTimingMiddleware> _logger;

    public RequestTimingMiddleware(
        RequestDelegate next,
        ILogger<RequestTimingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();

        // Register a callback that fires just before the response headers are sent.
        // This is necessary because once _next(context) returns,
        // the response may have already started streaming.
        context.Response.OnStarting(() =>
        {
            stopwatch.Stop();
            context.Response.Headers["X-Response-Time"] =
                $"{stopwatch.ElapsedMilliseconds}ms";
            return Task.CompletedTask;
        });

        try
        {
            await _next(context);
        }
        finally
        {
            stopwatch.Stop();
            _logger.LogInformation(
                "{Method} {Path} completed in {ElapsedMs}ms with status {StatusCode}",
                context.Request.Method,
                context.Request.Path,
                stopwatch.ElapsedMilliseconds,
                context.Response.StatusCode);
        }
    }
}

public static class RequestTimingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestTiming(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestTimingMiddleware>();
    }
}
```

> [!ad-note]
> `Response.OnStarting()` is used to set the header because by the time `await _next(context)` returns, the response headers may have already been sent to the client. `OnStarting` fires just before the headers are flushed, giving us the last opportunity to modify them.

> [!summary] Section Summary
> Request timing middleware wraps the `_next` call with a `Stopwatch`, uses `Response.OnStarting` to inject a header before the response is flushed, and logs the duration. This is a fundamental observability tool for any production API.

---

### Correlation ID Middleware

Generates or propagates a unique **correlation ID** for each request. This ID is used to correlate log entries across services in distributed systems.

```csharp
public class CorrelationIdMiddleware
{
    private const string CorrelationIdHeaderName = "X-Correlation-Id";
    private readonly RequestDelegate _next;
    private readonly ILogger<CorrelationIdMiddleware> _logger;

    public CorrelationIdMiddleware(
        RequestDelegate next,
        ILogger<CorrelationIdMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Check if the caller already provided a correlation ID
        // (common in service-to-service calls)
        if (!context.Request.Headers.TryGetValue(
                CorrelationIdHeaderName, out var correlationId)
            || string.IsNullOrWhiteSpace(correlationId))
        {
            correlationId = Guid.NewGuid().ToString("N");
        }

        // Store it in HttpContext.Items so downstream code can access it
        context.Items["CorrelationId"] = correlationId.ToString();

        // Add a log scope so all log entries include the correlation ID
        using (_logger.BeginScope(
            new Dictionary<string, object>
            {
                ["CorrelationId"] = correlationId.ToString()
            }))
        {
            _logger.LogInformation(
                "Request {Method} {Path} with CorrelationId {CorrelationId}",
                context.Request.Method,
                context.Request.Path,
                correlationId);

            // Echo the correlation ID back in the response
            context.Response.OnStarting(() =>
            {
                context.Response.Headers[CorrelationIdHeaderName] =
                    correlationId.ToString();
                return Task.CompletedTask;
            });

            await _next(context);
        }
    }
}

public static class CorrelationIdMiddlewareExtensions
{
    public static IApplicationBuilder UseCorrelationId(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<CorrelationIdMiddleware>();
    }
}
```

Usage:

```csharp
var app = builder.Build();

app.UseCorrelationId(); // Should be early in the pipeline
app.UseRequestTiming();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

Accessing the correlation ID in a controller:

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpGet("{orderId}")]
    public IActionResult GetOrder(int orderId)
    {
        var correlationId = HttpContext.Items["CorrelationId"]?.ToString();
        // Use correlationId in downstream service calls, logging, etc.
        return Ok(new { OrderId = orderId, CorrelationId = correlationId });
    }
}
```

> [!tip]
> Place correlation ID middleware as early as possible in the pipeline so that all subsequent middleware and handlers have access to the ID for logging and tracing.

> [!summary] Section Summary
> Correlation ID middleware checks for an existing `X-Correlation-Id` header, generates one if missing, stores it in `HttpContext.Items`, wraps downstream processing in a log scope, and echoes the ID back in the response. Essential for distributed tracing.

---

### API Key Authentication Middleware

Validates an **API key** from the `X-Api-Key` request header against a configured value. Rejects unauthorized requests before they reach controllers.

```csharp
public class ApiKeyMiddleware
{
    private const string ApiKeyHeaderName = "X-Api-Key";
    private readonly RequestDelegate _next;
    private readonly ILogger<ApiKeyMiddleware> _logger;
    private readonly string _configuredApiKey;

    public ApiKeyMiddleware(
        RequestDelegate next,
        ILogger<ApiKeyMiddleware> logger,
        IConfiguration configuration)
    {
        _next = next;
        _logger = logger;

        _configuredApiKey = configuration["Authentication:ApiKey"]
            ?? throw new InvalidOperationException(
                "Authentication:ApiKey is not configured in appsettings.json");
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Allow health check endpoints through without authentication
        if (context.Request.Path.StartsWithSegments("/health"))
        {
            await _next(context);
            return;
        }

        if (!context.Request.Headers.TryGetValue(
                ApiKeyHeaderName, out var extractedApiKey))
        {
            _logger.LogWarning(
                "API key missing from request {Method} {Path}",
                context.Request.Method,
                context.Request.Path);

            context.Response.StatusCode = StatusCodes.Status401Unauthorized;
            await context.Response.WriteAsJsonAsync(new
            {
                Error = "API key is required",
                Detail = $"Provide a valid API key in the {ApiKeyHeaderName} header"
            });
            return; // Short-circuit -- do NOT call _next
        }

        // Use a constant-time comparison to prevent timing attacks
        if (!CryptographicOperations.FixedTimeEquals(
                Encoding.UTF8.GetBytes(_configuredApiKey),
                Encoding.UTF8.GetBytes(extractedApiKey!)))
        {
            _logger.LogWarning(
                "Invalid API key provided for {Method} {Path}",
                context.Request.Method,
                context.Request.Path);

            context.Response.StatusCode = StatusCodes.Status403Forbidden;
            await context.Response.WriteAsJsonAsync(new
            {
                Error = "Invalid API key"
            });
            return; // Short-circuit
        }

        await _next(context); // Key is valid, continue the pipeline
    }
}

public static class ApiKeyMiddlewareExtensions
{
    public static IApplicationBuilder UseApiKeyAuthentication(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<ApiKeyMiddleware>();
    }
}
```

Configuration in `appsettings.json`:

```json
{
  "Authentication": {
    "ApiKey": "your-secret-api-key-here"
  }
}
```

> [!warning] Common Misconception
> String comparison with `==` is vulnerable to **timing attacks** -- an attacker can infer the correct key character by character based on response time differences. Always use `CryptographicOperations.FixedTimeEquals()` for secret comparisons. This method compares the entire byte sequence in constant time regardless of where a mismatch occurs.

> [!tip]
> For production APIs, consider using ASP.NET Core's built-in authentication system with a custom `AuthenticationHandler<T>` instead of middleware. The built-in system integrates with `[Authorize]` attributes, policies, and claims. API key middleware is appropriate for simple internal services.

> [!summary] Section Summary
> API key middleware extracts a key from the `X-Api-Key` header, compares it using constant-time comparison against a configured value, and short-circuits with 401/403 for invalid requests. It allows whitelisted paths (like health checks) through without authentication.

---

### Global Exception Handling Middleware

Catches unhandled exceptions from downstream middleware and controllers, logs them, and returns a standardized **ProblemDetails** JSON response per RFC 7807.

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;

public class GlobalExceptionHandlerMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionHandlerMiddleware> _logger;
    private readonly IHostEnvironment _environment;

    public GlobalExceptionHandlerMiddleware(
        RequestDelegate next,
        ILogger<GlobalExceptionHandlerMiddleware> logger,
        IHostEnvironment environment)
    {
        _next = next;
        _logger = logger;
        _environment = environment;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(
                ex,
                "Unhandled exception processing {Method} {Path}",
                context.Request.Method,
                context.Request.Path);

            await HandleExceptionAsync(context, ex);
        }
    }

    private async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        // Map exception types to appropriate HTTP status codes
        var (statusCode, title) = exception switch
        {
            ArgumentException =>
                (StatusCodes.Status400BadRequest, "Bad Request"),
            KeyNotFoundException =>
                (StatusCodes.Status404NotFound, "Resource Not Found"),
            UnauthorizedAccessException =>
                (StatusCodes.Status401Unauthorized, "Unauthorized"),
            InvalidOperationException =>
                (StatusCodes.Status409Conflict, "Conflict"),
            _ =>
                (StatusCodes.Status500InternalServerError, "Internal Server Error")
        };

        var problemDetails = new ProblemDetails
        {
            Status = statusCode,
            Title = title,
            Detail = _environment.IsDevelopment()
                ? exception.Message
                : "An error occurred while processing your request.",
            Instance = context.Request.Path,
            Type = $"https://httpstatuses.com/{statusCode}"
        };

        // Add trace ID for correlation with logs
        problemDetails.Extensions["traceId"] =
            context.TraceIdentifier;

        // Include stack trace only in development
        if (_environment.IsDevelopment())
        {
            problemDetails.Extensions["stackTrace"] =
                exception.StackTrace;
            problemDetails.Extensions["exceptionType"] =
                exception.GetType().FullName;
        }

        context.Response.StatusCode = statusCode;
        context.Response.ContentType = "application/problem+json";

        await context.Response.WriteAsJsonAsync(problemDetails);
    }
}

public static class GlobalExceptionHandlerMiddlewareExtensions
{
    public static IApplicationBuilder UseGlobalExceptionHandler(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<GlobalExceptionHandlerMiddleware>();
    }
}
```

> [!danger]
> Exception handling middleware must be placed **first** (or very early) in the pipeline. If it is placed after another middleware that throws, the exception will not be caught. The general rule: the exception handler wraps everything it needs to protect.

Usage:

```csharp
var app = builder.Build();

app.UseGlobalExceptionHandler(); // FIRST in the pipeline
app.UseCorrelationId();
app.UseRequestTiming();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

Example response for an unhandled `KeyNotFoundException`:

```json
{
  "type": "https://httpstatuses.com/404",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "Order with ID 12345 was not found.",
  "instance": "/api/orders/12345",
  "traceId": "0HN5LQVM8FJKP:00000001"
}
```

> [!warning] Common Misconception
> Developers sometimes check `context.Response.HasStarted` only to log a warning and then attempt to write anyway. Once `HasStarted` is `true`, you **cannot** modify the status code or headers. The response is already on the wire. Exception handling middleware works because it catches the exception before the response body is written. If your downstream middleware starts streaming a response and then throws, the exception handler cannot produce a clean ProblemDetails response.

> [!summary] Section Summary
> Global exception handling middleware wraps the entire downstream pipeline in a try-catch, maps exception types to HTTP status codes, and returns RFC 7807 ProblemDetails JSON. Include stack traces only in Development. Place this middleware first in the pipeline. Be aware that it cannot recover if the response has already started streaming.

---

## Testing Custom Middleware

Testing middleware involves creating a `DefaultHttpContext`, a mock `RequestDelegate`, invoking the middleware, and asserting on the response.

### Test Setup Pattern

```csharp
using Microsoft.AspNetCore.Http;
using Xunit;

public class RequestTimingMiddlewareTests
{
    private readonly RequestTimingMiddleware _middleware;
    private readonly DefaultHttpContext _httpContext;
    private bool _nextWasCalled;

    public RequestTimingMiddlewareTests()
    {
        _httpContext = new DefaultHttpContext();
        // DefaultHttpContext gives us an in-memory response body
        // but we need to set up a real stream for header testing
        _httpContext.Response.Body = new MemoryStream();
        _nextWasCalled = false;
    }

    private RequestDelegate CreateMockNext(int delayMs = 50)
    {
        return async context =>
        {
            _nextWasCalled = true;
            // Simulate some work
            await Task.Delay(delayMs);
            context.Response.StatusCode = StatusCodes.Status200OK;
        };
    }

    [Fact]
    public async Task InvokeAsync_SetsResponseTimeHeader()
    {
        // Arrange
        var logger = NullLogger<RequestTimingMiddleware>.Instance;
        var next = CreateMockNext(delayMs: 100);
        var middleware = new RequestTimingMiddleware(next, logger);

        _httpContext.Request.Method = "GET";
        _httpContext.Request.Path = "/api/orders";

        // Act
        await middleware.InvokeAsync(_httpContext);

        // Fire the OnStarting callbacks (simulate response being sent)
        // In integration tests, Kestrel does this automatically.
        // For unit tests with DefaultHttpContext, we must invoke them manually.
        await _httpContext.Response.StartAsync();

        // Assert
        Assert.True(_nextWasCalled, "Middleware should call next delegate");
        Assert.True(
            _httpContext.Response.Headers.ContainsKey("X-Response-Time"),
            "Response should contain X-Response-Time header");

        var headerValue = _httpContext.Response.Headers["X-Response-Time"].ToString();
        Assert.Matches(@"\d+ms", headerValue);
    }

    [Fact]
    public async Task InvokeAsync_CallsNextMiddleware()
    {
        // Arrange
        var logger = NullLogger<RequestTimingMiddleware>.Instance;
        var next = CreateMockNext();
        var middleware = new RequestTimingMiddleware(next, logger);

        // Act
        await middleware.InvokeAsync(_httpContext);

        // Assert
        Assert.True(_nextWasCalled);
    }

    [Fact]
    public async Task InvokeAsync_MeasuresTimeGreaterThanZero()
    {
        // Arrange
        var logger = NullLogger<RequestTimingMiddleware>.Instance;
        var next = CreateMockNext(delayMs: 200);
        var middleware = new RequestTimingMiddleware(next, logger);

        // Act
        await middleware.InvokeAsync(_httpContext);
        await _httpContext.Response.StartAsync();

        // Assert
        var headerValue = _httpContext.Response.Headers["X-Response-Time"].ToString();
        var ms = int.Parse(headerValue.Replace("ms", ""));
        Assert.True(ms >= 100, $"Expected at least 100ms, got {ms}ms");
    }
}
```

### Testing Middleware That Short-Circuits

```csharp
public class ApiKeyMiddlewareTests
{
    [Fact]
    public async Task InvokeAsync_MissingApiKey_Returns401()
    {
        // Arrange
        var context = new DefaultHttpContext();
        context.Response.Body = new MemoryStream();
        context.Request.Path = "/api/orders";

        var nextCalled = false;
        RequestDelegate next = _ =>
        {
            nextCalled = true;
            return Task.CompletedTask;
        };

        var config = new ConfigurationBuilder()
            .AddInMemoryCollection(new Dictionary<string, string?>
            {
                ["Authentication:ApiKey"] = "test-key-12345"
            })
            .Build();

        var logger = NullLogger<ApiKeyMiddleware>.Instance;
        var middleware = new ApiKeyMiddleware(next, logger, config);

        // Act
        await middleware.InvokeAsync(context);

        // Assert
        Assert.Equal(StatusCodes.Status401Unauthorized, context.Response.StatusCode);
        Assert.False(nextCalled, "Next middleware should NOT be called");
    }

    [Fact]
    public async Task InvokeAsync_ValidApiKey_CallsNext()
    {
        // Arrange
        var context = new DefaultHttpContext();
        context.Request.Path = "/api/orders";
        context.Request.Headers["X-Api-Key"] = "test-key-12345";

        var nextCalled = false;
        RequestDelegate next = _ =>
        {
            nextCalled = true;
            return Task.CompletedTask;
        };

        var config = new ConfigurationBuilder()
            .AddInMemoryCollection(new Dictionary<string, string?>
            {
                ["Authentication:ApiKey"] = "test-key-12345"
            })
            .Build();

        var logger = NullLogger<ApiKeyMiddleware>.Instance;
        var middleware = new ApiKeyMiddleware(next, logger, config);

        // Act
        await middleware.InvokeAsync(context);

        // Assert
        Assert.True(nextCalled, "Next middleware should be called for valid key");
    }

    [Fact]
    public async Task InvokeAsync_HealthEndpoint_BypassesAuthentication()
    {
        // Arrange
        var context = new DefaultHttpContext();
        context.Request.Path = "/health";
        // No API key header set

        var nextCalled = false;
        RequestDelegate next = _ =>
        {
            nextCalled = true;
            return Task.CompletedTask;
        };

        var config = new ConfigurationBuilder()
            .AddInMemoryCollection(new Dictionary<string, string?>
            {
                ["Authentication:ApiKey"] = "test-key-12345"
            })
            .Build();

        var logger = NullLogger<ApiKeyMiddleware>.Instance;
        var middleware = new ApiKeyMiddleware(next, logger, config);

        // Act
        await middleware.InvokeAsync(context);

        // Assert
        Assert.True(nextCalled, "Health endpoint should bypass API key check");
    }
}
```

> [!tip]
> For more realistic testing, use `WebApplicationFactory<T>` from `Microsoft.AspNetCore.Mvc.Testing` to create an integration test that exercises the full pipeline including middleware ordering, DI, and real HTTP semantics. Unit tests with `DefaultHttpContext` are faster but do not test the pipeline wiring.

> [!summary] Section Summary
> Test middleware by constructing a `DefaultHttpContext`, creating a mock `RequestDelegate` (a simple lambda), invoking `InvokeAsync`, and asserting on the response status code, headers, and whether `next` was called. Use `Response.StartAsync()` to trigger `OnStarting` callbacks in unit tests. For full pipeline testing, use `WebApplicationFactory<T>`.

---

## Middleware vs Action Filters

**Middleware** and **action filters** both intercept request processing, but they operate at different levels of the ASP.NET Core pipeline.

**Middleware** operates on the raw HTTP pipeline. It runs for every request -- including static files, health checks, SignalR, gRPC, and minimal API endpoints. It has no knowledge of MVC concepts like controllers, actions, or model binding.

**Action filters** operate inside the MVC/Razor Pages pipeline. They run only for requests that are routed to a controller action or Razor Page handler. They have access to MVC-specific context like `ActionExecutingContext`, model binding results, and action arguments.

| Aspect | Middleware | Action Filters |
|---|---|---|
| **Pipeline level** | HTTP pipeline (all requests) | MVC pipeline (controller actions only) |
| **Scope** | Every request including static files, gRPC, minimal APIs | Only requests routed to MVC controllers/Razor Pages |
| **Access to MVC context** | No -- only `HttpContext` | Yes -- `ActionExecutingContext`, model binding, action arguments |
| **DI support** | Constructor + `InvokeAsync` params | Constructor injection, `[ServiceFilter]`, `[TypeFilter]` |
| **Ordering** | Pipeline order (first registered = outermost) | Filter order (global, controller, action levels) |
| **Short-circuiting** | Skip `next()` to short-circuit entire pipeline | Set `context.Result` to short-circuit action execution |
| **Best for** | Cross-cutting concerns: logging, CORS, timing, error handling, correlation IDs, compression | MVC-specific concerns: validation, authorization attributes, caching, result transformation |

### Decision Guide

Use **middleware** when:
- The logic applies to all requests regardless of endpoint type
- You need to run before routing or authentication
- You are dealing with raw HTTP concerns (headers, status codes, request body)
- You want to affect non-MVC endpoints (minimal APIs, gRPC, static files)

Use **action filters** when:
- The logic is specific to MVC controller actions
- You need access to action arguments, model binding, or `ActionResult`
- You want to apply the filter selectively via attributes (`[ServiceFilter]`, `[TypeFilter]`)
- The behavior is closely tied to business logic rather than infrastructure

> [!example]
> **Request timing**: Use middleware -- you want to time all requests, not just MVC actions.
> **Input validation logging**: Use an action filter -- you need access to the model binding result.
> **API key check**: Can be either, but middleware is better if you also serve minimal API endpoints.
> **Audit logging of action parameters**: Use an action filter -- you need `ActionExecutingContext.ActionArguments`.

> [!warning] Common Misconception
> Developers sometimes put cross-cutting concerns like exception handling into an `IExceptionFilter` and assume it catches all exceptions. It does not -- `IExceptionFilter` only catches exceptions thrown during action execution and result execution within the MVC pipeline. Exceptions thrown in middleware (before the MVC pipeline) or in non-MVC endpoints will bypass it entirely. Use exception handling middleware for truly global coverage.

> [!summary] Section Summary
> Middleware runs for all HTTP requests at the pipeline level; action filters run only for MVC controller actions inside the MVC pipeline. Use middleware for infrastructure cross-cutting concerns (logging, timing, error handling) and action filters for MVC-specific concerns (validation, authorization attributes, result transformation). Exception handling should be middleware for full coverage.

---

## Related Topics

- [[Middleware Overview]] -- how the ASP.NET Core request pipeline works
- [[Request Pipeline]] -- middleware ordering, branching with `Map`, and terminal middleware
- [[Built-in Middleware]] -- authentication, CORS, static files, response compression
- [[Service Lifetimes]] -- singleton, scoped, transient and how they interact with middleware
- [[DI Overview]] -- the ASP.NET Core dependency injection container
- [[Error Handling and Logging]] -- structured logging, exception pages, ProblemDetails

---

## Further Reading

- [[Action Filters]] -- MVC-specific request/response interception
- [[Minimal APIs]] -- middleware works with minimal API endpoints too
- [[Options Pattern]] -- configuring middleware with `IOptions<T>`
- [[Integration Testing]] -- using `WebApplicationFactory<T>` to test the full pipeline
- [[Response Caching]] -- built-in middleware for HTTP caching
- [[Rate Limiting]] -- built-in rate limiting middleware in .NET 7+

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Custom middleware** extends the ASP.NET Core [[Request Pipeline]] with user-defined logic that runs for every HTTP request.
>
> **Three approaches exist:**
> 1. **Inline** (`app.Use()`) -- quick lambdas for prototyping
> 2. **Convention-based** -- a class with `RequestDelegate` constructor and `InvokeAsync` method; singleton lifetime; the standard production approach
> 3. **Factory-based** (`IMiddleware`) -- resolved from DI per request; supports scoped constructor injection; requires explicit DI registration
>
> **Dependency injection** in convention-based middleware splits into constructor injection (singleton services only) and method injection via `InvokeAsync` parameters (any lifetime). Injecting scoped services like `DbContext` into the constructor is a common bug -- the singleton middleware captures a single instance that outlives its scope.
>
> **The extension method pattern** (`app.UseRequestTiming()`) wraps `UseMiddleware<T>()` for clean, discoverable registration following ASP.NET Core conventions.
>
> **Real-world examples** include request timing (Stopwatch + `OnStarting` callback), correlation ID propagation (check/generate/echo header), API key validation (constant-time comparison, short-circuit on failure), and global exception handling (try-catch + ProblemDetails JSON response).
>
> **Testing** uses `DefaultHttpContext` and a mock `RequestDelegate` lambda. Call `Response.StartAsync()` to trigger `OnStarting` callbacks in unit tests. Use `WebApplicationFactory<T>` for integration tests.
>
> **Middleware vs action filters**: middleware runs at the HTTP pipeline level for all requests; action filters run inside the MVC pipeline for controller actions only. Use middleware for cross-cutting infrastructure concerns, action filters for MVC-specific logic.
