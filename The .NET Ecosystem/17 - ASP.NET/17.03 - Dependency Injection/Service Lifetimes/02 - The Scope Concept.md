---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - lifetimes
---

## The Scope Concept

### What Is a Scope?

A scope is a container boundary that controls the lifetime of scoped services. When a scope is created, any scoped service resolved within it returns the same instance. When the scope is disposed, all scoped services within it are also disposed.

> [!info] Definition
> An `IServiceScope` is a disposable object that wraps an `IServiceProvider`. Services resolved from this scoped provider follow scoped lifetime rules -- one instance per scope. The scope is created by `IServiceScopeFactory.CreateScope()`.

### How HTTP Requests Get a Scope

ASP.NET Core's middleware pipeline automatically creates a scope for each incoming HTTP request. You do not need to do this yourself for normal request handling. The flow looks like this:

```
HTTP Request Arrives
    --> Middleware pipeline creates IServiceScope
        --> Controller and services resolved from this scope
        --> Scoped services shared within this request
    --> Response sent
    --> Scope disposed (all scoped services disposed)
```

This is handled internally by the framework's `RequestServicesFeature`. You never see it, but it is running on every request.

### Creating Manual Scopes

For background tasks, hosted services, or any code running outside the HTTP request pipeline, there is no automatic scope. You must create one manually using `IServiceScopeFactory`:

```csharp
public class OrderProcessingBackgroundService : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;

    public OrderProcessingBackgroundService(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            // Create a new scope for each iteration
            using (var scope = _scopeFactory.CreateScope())
            {
                var orderRepo = scope.ServiceProvider
                    .GetRequiredService<IOrderRepository>();
                
                var dbContext = scope.ServiceProvider
                    .GetRequiredService<AppDbContext>();

                await ProcessPendingOrdersAsync(orderRepo, dbContext);
            }
            // Scope disposed here -- DbContext, OrderRepository cleaned up

            await Task.Delay(TimeSpan.FromMinutes(1), stoppingToken);
        }
    }
}
```

> [!tip]
> Always create a new scope for each logical unit of work in background services. If you reuse the same scope for hours, the DbContext change tracker accumulates entities and memory grows indefinitely. Each `using` block gives you a fresh DbContext.

> [!ad-note]
> `IServiceScopeFactory` is itself registered as a Singleton, so it is safe to inject into other singletons like `BackgroundService`. This is the correct way for singletons to access scoped services -- not by injecting them directly.

> [!summary] Section Summary
> - A scope is a container boundary that controls the lifetime of scoped services.
> - ASP.NET Core automatically creates a scope per HTTP request via the middleware pipeline.
> - Background services and hosted services have no automatic scope -- you must create one manually.
> - `IServiceScopeFactory.CreateScope()` creates a new scope with its own `IServiceProvider`.
> - Always dispose scopes when done to clean up scoped services like DbContext.
