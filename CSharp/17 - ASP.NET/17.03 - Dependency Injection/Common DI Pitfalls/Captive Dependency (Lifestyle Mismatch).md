---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - pitfalls
---

## Captive Dependency (Lifestyle Mismatch)

> [!info] Definition
> A **captive dependency** occurs when a longer-lived service (typically a Singleton) depends on a shorter-lived service (typically Scoped or Transient). The shorter-lived service becomes "captive" -- it is captured by the Singleton and forced to live as long as the Singleton does, which is the entire application lifetime.

This is the single most dangerous DI pitfall in ASP.NET Core because it silently corrupts your application's behavior without any obvious error (unless you enable scope validation).

### The Hierarchy Rule

The [[Service Lifetimes]] hierarchy dictates which services can safely depend on which:

| Parent Lifetime | Can Depend On |
|---|---|
| Transient | Transient, Scoped, Singleton |
| Scoped | Scoped, Singleton |
| Singleton | **Singleton ONLY** |

The rule is simple: **a service can only depend on services with an equal or longer lifetime.**

### The Buggy Code

Consider a reporting service registered as a Singleton that needs to query orders from the database:

```csharp
// Registration in Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString)); // DbContext is Scoped by default

builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddSingleton<IReportGenerator, ReportGenerator>(); // DANGER
```

```csharp
// ReportGenerator.cs -- THE BUG
public class ReportGenerator : IReportGenerator
{
    private readonly IOrderRepository _orderRepository;

    // This constructor injection creates a captive dependency.
    // The Singleton ReportGenerator captures the Scoped IOrderRepository,
    // which in turn captures the Scoped DbContext.
    public ReportGenerator(IOrderRepository orderRepository)
    {
        _orderRepository = orderRepository;
    }

    public SalesReport GenerateMonthlySalesReport(int month, int year)
    {
        // This repository instance was created ONCE when the Singleton was resolved.
        // It uses a DbContext that was also created once and never disposed.
        var orders = _orderRepository.GetOrdersByMonth(month, year);
        return new SalesReport(orders);
    }
}
```

### What Goes Wrong

When the application starts and `ReportGenerator` is first resolved:

1. The container creates a single `ReportGenerator` instance (Singleton behavior)
2. To satisfy its constructor, the container creates an `OrderRepository` (which should be Scoped)
3. The `OrderRepository` gets a `DbContext` (also Scoped)
4. All three objects now live **forever** -- for the entire lifetime of the application

The consequences are severe:

- **Stale data**: The `DbContext` uses a first-level cache (the Change Tracker). After the first query, subsequent queries may return cached entities instead of fresh data from the database. Changes made by other requests are invisible.
- **Memory leaks**: The Change Tracker accumulates every entity ever queried and never releases them. Over hours or days, memory usage grows unbounded.
- **Connection pool exhaustion**: The captured `DbContext` holds its database connection open indefinitely. Under load, other requests cannot obtain connections from the pool.
- **Concurrency exceptions**: If two HTTP requests simultaneously hit the `ReportGenerator`, they share the same `DbContext` instance. `DbContext` is **not thread-safe** -- this causes `InvalidOperationException` or corrupted data.

> [!danger] Silent Corruption
> The application does not crash immediately. It starts correctly, serves the first few requests fine, and then gradually degrades. Stale data appears intermittently. Memory creeps up. Eventually, under load, you get cryptic `DbContext` concurrency exceptions or connection timeouts. This makes captive dependencies extremely hard to diagnose in production.

### The Fix: IServiceScopeFactory

The correct approach is to create a new scope each time the Singleton needs to use the Scoped service:

```csharp
// ReportGenerator.cs -- THE FIX
public class ReportGenerator : IReportGenerator
{
    private readonly IServiceScopeFactory _scopeFactory;

    public ReportGenerator(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }

    public SalesReport GenerateMonthlySalesReport(int month, int year)
    {
        // Create a new scope each time we need the Scoped service.
        // The scope (and everything resolved from it) is disposed at the end of the using block.
        using var scope = _scopeFactory.CreateScope();
        var orderRepository = scope.ServiceProvider.GetRequiredService<IOrderRepository>();

        var orders = orderRepository.GetOrdersByMonth(month, year);
        return new SalesReport(orders);
    }
}
```

Now each call to `GenerateMonthlySalesReport` gets a fresh `OrderRepository` with a fresh `DbContext`. When the `using` block ends, both are properly disposed. No stale data, no memory leaks, no connection hoarding.

> [!ad-note]
> `IServiceScopeFactory` is itself a Singleton, so it is safe to inject into another Singleton. This is the standard pattern for Singleton services that need access to Scoped services -- you will see it heavily in `IHostedService` / `BackgroundService` implementations.

### Catching It Early with ValidateScopes

ASP.NET Core provides a built-in safety net. In the Development environment, `ValidateScopes` is enabled by default on the default host builder:

```csharp
// This is the default behavior in Development -- you don't need to add this manually
var builder = WebApplication.CreateBuilder(args);
// ValidateScopes = true in Development
// ValidateOnBuild = true in Development
```

With scope validation enabled, if you try to resolve a Scoped service from the root provider (as happens with a captive dependency), you get an immediate exception at startup:

```
System.InvalidOperationException: Cannot resolve scoped service
'IOrderRepository' from root provider.
```

> [!warning] Common Misconception
> `ValidateScopes` is only enabled in the Development environment by default. In Production, captive dependencies are **not** detected and will silently cause the problems described above. Always test thoroughly in Development before deploying. You can also enable it in Production for extra safety at a small performance cost:
> ```csharp
> builder.Host.UseDefaultServiceProvider(options =>
> {
>     options.ValidateScopes = true;
>     options.ValidateOnBuild = true;
> });
> ```

> [!summary] Section Summary
> - A captive dependency occurs when a Singleton captures a Scoped service, forcing it to live forever
> - The captured Scoped service (and its DbContext) becomes stale, leaks memory, and is not thread-safe
> - Fix by injecting `IServiceScopeFactory` and creating a new scope each time you need the Scoped service
> - `ValidateScopes` catches this at startup in Development but is disabled in Production by default
> - This is the most common and most dangerous DI pitfall in ASP.NET Core
