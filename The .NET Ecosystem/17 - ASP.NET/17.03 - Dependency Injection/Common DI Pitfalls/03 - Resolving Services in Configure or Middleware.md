---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - pitfalls
---

## Resolving Services in Configure or Middleware

A common mistake is resolving Scoped services directly from `app.Services` (the root `IServiceProvider`) during application configuration. This creates the same problem as a captive dependency.

### The Problem

```csharp
// Program.cs -- BAD: resolving Scoped service from root provider
var app = builder.Build();

// app.Services is the ROOT service provider -- there is no scope here.
// If IOrderService is registered as Scoped, this creates a root-scoped instance
// that lives forever, just like a captive dependency.
var orderService = app.Services.GetRequiredService<IOrderService>(); // BAD
orderService.SeedDefaultOrders();
```

```csharp
// Program.cs -- BAD: middleware constructor captures Scoped service
app.Use(async (context, next) =>
{
    // This lambda closes over 'orderService' resolved from the root provider.
    // Same root-scoped instance for every single request.
    var report = orderService.GetDailyReport();
    context.Response.Headers.Append("X-Orders-Today", report.Count.ToString());
    await next();
});
```

### The Fix: Create a Scope Explicitly

```csharp
// Program.cs -- CORRECT: create a scope for startup operations
var app = builder.Build();

using (var scope = app.Services.CreateScope())
{
    var orderService = scope.ServiceProvider.GetRequiredService<IOrderService>();
    orderService.SeedDefaultOrders();
} // Scope is disposed here -- the Scoped service is properly cleaned up
```

### The Fix: Use HttpContext.RequestServices in Middleware

```csharp
// Approach 1: Resolve from the request scope via HttpContext
app.Use(async (context, next) =>
{
    // HttpContext.RequestServices is scoped to this specific HTTP request.
    // This gives you a properly scoped instance.
    var orderService = context.RequestServices.GetRequiredService<IOrderService>();
    var report = orderService.GetDailyReport();
    context.Response.Headers.Append("X-Orders-Today", report.Count.ToString());
    await next();
});
```

```csharp
// Approach 2: Inject Scoped services into the InvokeAsync method (convention-based middleware)
public class DailyReportMiddleware
{
    private readonly RequestDelegate _next;

    // Singleton dependencies in the constructor (resolved once)
    public DailyReportMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    // Scoped dependencies as method parameters (resolved per request)
    public async Task InvokeAsync(HttpContext context, IOrderService orderService)
    {
        var report = orderService.GetDailyReport();
        context.Response.Headers.Append("X-Orders-Today", report.Count.ToString());
        await _next(context);
    }
}
```

> [!ad-note]
> Convention-based middleware classes have their **constructor** called once (like a Singleton), but their `InvokeAsync` method is called per request. ASP.NET Core resolves the method parameters from the request scope, making it safe to accept Scoped services as method parameters but NOT as constructor parameters. See [[The .NET Ecosystem/17 - ASP.NET/17.03 - Dependency Injection/DI Overview/DI Overview]] for more on middleware DI.

> [!summary] Section Summary
> - `app.Services` is the root `IServiceProvider` -- resolving Scoped services from it creates a root-scoped instance that never disposes
> - For startup/seeding operations, wrap the resolution in `app.Services.CreateScope()`
> - In middleware, use `HttpContext.RequestServices` or accept Scoped services as `InvokeAsync` method parameters
> - Never inject Scoped services into a middleware constructor -- middleware constructors are resolved once like Singletons
> - This is fundamentally the same bug as a captive dependency, just in a different location
