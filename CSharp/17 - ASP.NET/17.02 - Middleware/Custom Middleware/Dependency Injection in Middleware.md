---
tags:
  - csharp
  - asp-net-core
  - middleware
  - custom
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
