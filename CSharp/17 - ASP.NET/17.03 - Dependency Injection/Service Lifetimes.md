---
title: Service Lifetimes
date: 2026-06-18
tags: [csharp, asp-net-core, dependency-injection, lifetimes]
aliases: [DI Lifetimes, Service Lifetime, Transient Scoped Singleton]
status: complete
---

# Service Lifetimes

> [!abstract] Overview
> ASP.NET Core's dependency injection container manages how long service instances live through three distinct lifetimes: **Transient**, **Scoped**, and **Singleton**. Choosing the correct lifetime for each service is critical -- it affects memory usage, thread safety, data consistency, and can introduce subtle bugs like captive dependencies. This note covers each lifetime in depth, demonstrates their behavior with concrete code, and addresses the most common pitfalls developers encounter.

---

## Table of Contents

- [The Three Lifetimes](#the-three-lifetimes)
- [Transient](#transient)
- [Scoped](#scoped)
- [Singleton](#singleton)
- [Comparison Table](#comparison-table)
- [The Scope Concept](#the-scope-concept)
- [Why DbContext Should Be Scoped](#why-dbcontext-should-be-scoped)
- [Why HttpClient Uses Singleton Registration](#why-httpclient-uses-singleton-registration)
- [Demonstrating Lifetime Behavior](#demonstrating-lifetime-behavior)
- [The Captive Dependency Problem](#the-captive-dependency-problem)
- [ValidateScopes and ValidateOnBuild](#validatescopes-and-validateonbuild)
- [Comprehensive Summary](#comprehensive-summary)
- [Related Topics](#related-topics)
- [Further Reading](#further-reading)

---

## The Three Lifetimes

When you register a service with the ASP.NET Core DI container, you must choose one of three lifetimes. Each lifetime answers a single question: **when does the container create a new instance, and when does it reuse an existing one?**

Think of it with this mental model:

| Lifetime | Mental Model |
|---|---|
| **Transient** | A vending machine -- every button press gives you a fresh item, even if you press the same button twice in a row |
| **Scoped** | A whiteboard in a meeting room -- everyone in the same meeting (request) shares it, but the next meeting gets a clean board |
| **Singleton** | A wall clock in the office -- there is exactly one, everyone sees the same clock, it exists for the life of the building |

All three are registered through `IServiceCollection` in `Program.cs` (or `Startup.cs` in older project styles):

```csharp
var builder = WebApplication.CreateBuilder(args);

// Transient -- new instance every time
builder.Services.AddTransient<IOrderValidator, OrderValidator>();

// Scoped -- one instance per HTTP request
builder.Services.AddScoped<IOrderRepository, OrderRepository>();

// Singleton -- one instance for the entire app
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
```

> [!info] Definition
> A **service lifetime** governs how the DI container creates, reuses, and disposes of service instances. It is set at registration time and cannot be changed at runtime.

> [!summary] Section Summary
> - ASP.NET Core offers three service lifetimes: Transient, Scoped, and Singleton.
> - The lifetime is chosen at registration time via `AddTransient`, `AddScoped`, or `AddSingleton`.
> - Each lifetime controls instance creation, reuse, and disposal differently.
> - Choosing the wrong lifetime leads to bugs around stale data, thread safety, and memory leaks.
> - The mental models (vending machine, whiteboard, wall clock) help anchor the differences.

---

## Transient

**Transient** services are created every single time they are requested from the container. No caching, no reuse -- every injection point gets a brand-new instance.

### Registration

```csharp
builder.Services.AddTransient<IOrderValidator, OrderValidator>();
```

### Behavior

Even within the same HTTP request, if two different classes both depend on `IOrderValidator`, they each receive a **different** instance:

```csharp
public class OrderController : ControllerBase
{
    private readonly IOrderValidator _validator; // Instance A

    public OrderController(IOrderValidator validator)
    {
        _validator = validator;
    }
}

public class OrderService
{
    private readonly IOrderValidator _validator; // Instance B (different from A!)

    public OrderService(IOrderValidator validator)
    {
        _validator = validator;
    }
}
```

If both `OrderController` and `OrderService` are resolved during the same request, `_validator` in each class points to a **separate** `OrderValidator` instance.

### When to Use Transient

Transient is ideal for:
- **Lightweight, stateless services** -- validators, formatters, mapping services
- **Services with no shared mutable state** -- each consumer gets its own clean copy
- **Services that are cheap to construct** -- since a new one is created every time

```csharp
// Good candidates for Transient
builder.Services.AddTransient<IOrderValidator, OrderValidator>();
builder.Services.AddTransient<IAddressFormatter, AddressFormatter>();
builder.Services.AddTransient<ICustomerMapper, CustomerMapper>();
```

> [!warning] Common Misconception
> "Transient means one instance per request." This is wrong. Transient means one instance **per injection**. A single request that injects the same transient service in three places will create three separate instances. If you want one-per-request, use **Scoped**.

> [!ad-note]
> Transient services that implement `IDisposable` are tracked by the container and disposed when the scope (request) ends. This means transient disposable services still have their `Dispose()` called -- but many transient instances may accumulate within a single request, increasing memory pressure.

> [!summary] Section Summary
> - Transient creates a new instance for every injection point, every time.
> - Even within the same HTTP request, different constructor parameters get different instances.
> - Best suited for lightweight, stateless, cheap-to-create services.
> - Disposable transient services are tracked and disposed at scope end, but many instances can accumulate.
> - Do not confuse Transient with "one per request" -- that is Scoped.

---

## Scoped

**Scoped** services are created once per scope. In ASP.NET Core, each HTTP request automatically gets its own scope, so scoped services are effectively **one instance per HTTP request**.

### Registration

```csharp
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
```

### Behavior

Within a single request, every class that asks for `IOrderRepository` gets the **same** instance:

```csharp
public class OrderController : ControllerBase
{
    private readonly IOrderRepository _repo; // Instance A

    public OrderController(IOrderRepository repo)
    {
        _repo = repo;
    }
}

public class OrderService
{
    private readonly IOrderRepository _repo; // Also Instance A (same!)

    public OrderService(IOrderRepository repo)
    {
        _repo = repo;
    }
}
```

But a **different** HTTP request creates a completely new scope and therefore a new instance:

```
Request 1: OrderRepository instance #1 (shared within request 1)
Request 2: OrderRepository instance #2 (shared within request 2)
Request 3: OrderRepository instance #3 (shared within request 3)
```

### When to Use Scoped

Scoped is ideal for:
- **EF Core DbContext** -- the most common scoped service
- **Unit of Work** implementations
- **Request-specific state** -- anything tied to the current user or operation
- **Services that need to share state within a single request** but be isolated between requests

```csharp
// Good candidates for Scoped
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<ICurrentUserService, CurrentUserService>();
```

> [!tip]
> Scoped is the "default safe choice" for most business services. If you are unsure, start with Scoped -- it prevents cross-request data leaks (unlike Singleton) while avoiding unnecessary instance creation (unlike Transient).

> [!ad-note]
> Scoped services are disposed at the end of the request when the scope is disposed. This is why `DbContext` works well as scoped -- its connections and change tracker are cleaned up automatically after each request.

> [!summary] Section Summary
> - Scoped creates one instance per scope (one per HTTP request in ASP.NET Core).
> - All classes within the same request share the same scoped instance.
> - Different requests always get different instances.
> - Ideal for DbContext, Unit of Work, and request-specific state.
> - Scoped services are disposed when the scope (request) ends.

---

## Singleton

**Singleton** services are created once on first request and the same instance is reused for the entire lifetime of the application. Every request, every user, every thread -- they all share the same instance.

### Registration

```csharp
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
```

You can also register a singleton with a pre-built instance:

```csharp
var cacheService = new MemoryCacheService();
builder.Services.AddSingleton<ICacheService>(cacheService);
```

Or with a factory:

```csharp
builder.Services.AddSingleton<ICacheService>(sp =>
{
    var config = sp.GetRequiredService<IConfiguration>();
    return new MemoryCacheService(config["CacheSize"]);
});
```

### Behavior

The container creates the instance when it is first requested (lazy initialization). From that point on, the same instance is returned everywhere:

```
Request 1: MemoryCacheService instance #1
Request 2: MemoryCacheService instance #1 (same!)
Request 3: MemoryCacheService instance #1 (same!)
... all the way until the application shuts down
```

### When to Use Singleton

Singleton is ideal for:
- **In-memory caches** -- shared across all requests
- **Configuration wrappers** -- application-wide settings that do not change per request
- **HttpClient factories** -- managed through `IHttpClientFactory`
- **Logging infrastructure** -- shared loggers
- **Services with expensive initialization** -- created once, amortized across all requests

```csharp
// Good candidates for Singleton
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
builder.Services.AddSingleton<IAppConfiguration, AppConfiguration>();
builder.Services.AddSingleton<IEmailTemplateLoader, EmailTemplateLoader>();
```

> [!danger]
> Singleton services **must be thread-safe**. Multiple requests (threads) will access the same instance concurrently. If your singleton modifies shared state without synchronization, you will get race conditions and data corruption. Use `ConcurrentDictionary`, `lock`, `SemaphoreSlim`, or immutable data structures.

> [!warning] Common Misconception
> "I can inject a scoped service (like `DbContext`) into a singleton." You cannot do this safely. The scoped service would be resolved once during the singleton's creation and then held forever -- it becomes a **captive dependency**. See [[#The Captive Dependency Problem]].

> [!summary] Section Summary
> - Singleton creates one instance for the entire application lifetime, shared across all requests and users.
> - Created lazily on first request (or eagerly if you provide a pre-built instance).
> - Must be thread-safe since concurrent requests share the same instance.
> - Ideal for caches, configuration, and expensive-to-create services.
> - Never inject scoped or transient services directly into a singleton.

---

## Comparison Table

| Lifetime | Created When | Disposed When | Same Within Request? | Same Across Requests? | Thread-Safe Required? | Typical Use Cases |
|---|---|---|---|---|---|---|
| **Transient** | Every time it is injected | When the scope (request) ends | No -- each injection gets a new instance | No | No (each consumer gets its own) | Validators, formatters, mappers, lightweight stateless services |
| **Scoped** | Once per scope (request) | When the scope (request) ends | Yes -- all injections share one instance | No -- each request gets its own | Only if used across async calls within the same request | DbContext, Unit of Work, repositories, request-specific state |
| **Singleton** | Once (on first request) | When the application shuts down | Yes | Yes -- same instance forever | Yes -- multiple threads access it concurrently | Caches, configuration, HttpClientFactory, loggers |

> [!ad-note]
> The "Disposed When" column only applies to services implementing `IDisposable` or `IAsyncDisposable`. The container calls `Dispose()` automatically at the appropriate time for each lifetime.

> [!summary] Section Summary
> - Transient: new per injection, disposed per scope, no thread safety needed.
> - Scoped: new per request, disposed per request, shared within request.
> - Singleton: created once, disposed at shutdown, shared everywhere, thread safety required.
> - The comparison table serves as a quick reference when choosing a lifetime.
> - Disposal is handled automatically by the container for all three lifetimes.

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

---

## Why DbContext Should Be Scoped

EF Core registers `DbContext` as scoped by default when you call `AddDbContext<T>()`. This is not arbitrary -- it is the only safe choice for most applications.

### Why Not Singleton?

A singleton `DbContext` would be shared across all requests and all threads simultaneously. This causes multiple serious problems:

1. **DbContext is not thread-safe.** Concurrent requests modifying the same `DbContext` instance will throw exceptions or corrupt data.
2. **The change tracker grows forever.** Every entity loaded by any request stays tracked until the application restarts, consuming ever-increasing memory.
3. **Stale data.** Entities tracked from an earlier request do not reflect database changes made by other requests or external systems.
4. **Connection management.** A singleton context may hold database connections open far longer than necessary.

### Why Not Transient?

A transient `DbContext` gives a new instance to every injection point. Within a single request, this means:

1. **Inconsistent reads.** `OrderService` and `InventoryService` within the same request have different contexts. They may see different snapshots of the database.
2. **No shared transactions.** You cannot wrap operations across multiple services in a single transaction because they each have their own `DbContext` and therefore their own connection.
3. **Change tracker isolation.** An entity loaded in one service is unknown to another service's context, making cross-service operations awkward.

### Why Scoped Is Correct

Scoped gives you one `DbContext` per request:

- **Thread safety is handled** -- a single request is processed sequentially (or with controlled async/await), so the context is only accessed by one logical thread.
- **Change tracker is bounded** -- it accumulates entities only for the duration of one request, then gets disposed.
- **Shared state within a request** -- all services in the same request share the same context, enabling transactions and consistent reads.
- **Automatic disposal** -- the context is disposed when the request scope ends.

```csharp
// EF Core's default registration -- scoped
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));

// This is equivalent to:
builder.Services.AddScoped<AppDbContext>(sp =>
{
    var options = sp.GetRequiredService<DbContextOptions<AppDbContext>>();
    return new AppDbContext(options);
});
```

> [!danger]
> Never change `DbContext` to Singleton. It will appear to work in development with low traffic, then fail catastrophically under concurrent production load with `InvalidOperationException` or silent data corruption.

> [!summary] Section Summary
> - DbContext is registered as Scoped by default via `AddDbContext<T>()`.
> - Singleton DbContext breaks thread safety, leaks memory via the change tracker, and serves stale data.
> - Transient DbContext prevents shared transactions and causes inconsistent reads within a request.
> - Scoped DbContext provides one instance per request: thread-safe, bounded, and shared across services.
> - Never change DbContext's lifetime without understanding the full consequences.

---

## Why HttpClient Uses Singleton Registration

### The Socket Exhaustion Problem

Creating `HttpClient` instances directly (or registering them as Transient) causes a well-known problem: **socket exhaustion**. Each `HttpClient` instance manages its own connection pool. When you dispose an `HttpClient`, the underlying sockets enter a `TIME_WAIT` state and are not immediately available for reuse. Under load, this exhausts the available sockets on the machine.

```csharp
// BAD: Do not do this
public class OrderApiClient
{
    public async Task<Order> GetOrderAsync(int orderId)
    {
        // Creates a new HttpClient (and connection pool) every call!
        using var client = new HttpClient();
        var response = await client.GetAsync($"https://api.example.com/orders/{orderId}");
        return await response.Content.ReadFromJsonAsync<Order>();
    }
}
```

### The Solution: IHttpClientFactory

`IHttpClientFactory` manages a pool of `HttpMessageHandler` instances (the underlying connection handlers), reusing them across `HttpClient` instances. The `HttpClient` instances themselves are transient (lightweight wrappers), but the handlers are pooled and recycled.

```csharp
// Registration in Program.cs
builder.Services.AddHttpClient<IOrderApiClient, OrderApiClient>(client =>
{
    client.BaseAddress = new Uri("https://api.example.com");
    client.DefaultRequestHeaders.Add("Accept", "application/json");
    client.Timeout = TimeSpan.FromSeconds(30);
});
```

```csharp
// The typed client
public class OrderApiClient : IOrderApiClient
{
    private readonly HttpClient _client;

    // HttpClient is injected by the factory -- handler is pooled
    public OrderApiClient(HttpClient client)
    {
        _client = client;
    }

    public async Task<Order> GetOrderAsync(int orderId)
    {
        var response = await _client.GetAsync($"/orders/{orderId}");
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<Order>();
    }
}
```

> [!ad-note]
> When you use `AddHttpClient<TClient, TImplementation>()`, the typed client (`OrderApiClient`) is registered as **Transient** with the container, but the underlying `HttpMessageHandler` is managed by the factory with a default lifetime of 2 minutes before recycling. This gives you the best of both worlds: fresh client instances (no stale state) with pooled connections (no socket exhaustion).

> [!tip]
> Always use `IHttpClientFactory` (via `AddHttpClient`) instead of `new HttpClient()`. This applies to all HTTP calls in ASP.NET Core, whether to external APIs, microservices, or third-party providers. See [[Registration Patterns]] for more factory patterns.

> [!summary] Section Summary
> - Creating `HttpClient` directly or as Transient causes socket exhaustion under load.
> - `IHttpClientFactory` pools `HttpMessageHandler` instances while keeping `HttpClient` wrappers transient.
> - Use `AddHttpClient<TClient, TImplementation>()` for typed clients with pooled connections.
> - The default handler lifetime is 2 minutes before recycling, balancing connection reuse and DNS changes.
> - Never use `new HttpClient()` in a constructor or method -- always go through the factory.

---

## Demonstrating Lifetime Behavior

The best way to understand lifetimes is to see them in action. Here is a complete example that makes the differences visible.

### The Service

```csharp
public interface ILifetimeReporter
{
    Guid InstanceId { get; }
    string Lifetime { get; }
}

public class TransientReporter : ILifetimeReporter
{
    public Guid InstanceId { get; } = Guid.NewGuid();
    public string Lifetime => "Transient";
}

public class ScopedReporter : ILifetimeReporter
{
    public Guid InstanceId { get; } = Guid.NewGuid();
    public string Lifetime => "Scoped";
}

public class SingletonReporter : ILifetimeReporter
{
    public Guid InstanceId { get; } = Guid.NewGuid();
    public string Lifetime => "Singleton";
}
```

### Registration

```csharp
builder.Services.AddTransient<TransientReporter>();
builder.Services.AddScoped<ScopedReporter>();
builder.Services.AddSingleton<SingletonReporter>();
builder.Services.AddTransient<ChildService>();
```

### A Child Service (to show same-request behavior)

```csharp
public class ChildService
{
    public TransientReporter Transient { get; }
    public ScopedReporter Scoped { get; }
    public SingletonReporter Singleton { get; }

    public ChildService(
        TransientReporter transient,
        ScopedReporter scoped,
        SingletonReporter singleton)
    {
        Transient = transient;
        Scoped = scoped;
        Singleton = singleton;
    }
}
```

### The Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class LifetimeController : ControllerBase
{
    private readonly TransientReporter _transient;
    private readonly ScopedReporter _scoped;
    private readonly SingletonReporter _singleton;
    private readonly ChildService _childService;

    public LifetimeController(
        TransientReporter transient,
        ScopedReporter scoped,
        SingletonReporter singleton,
        ChildService childService)
    {
        _transient = transient;
        _scoped = scoped;
        _singleton = singleton;
        _childService = childService;
    }

    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new
        {
            Controller = new
            {
                Transient = _transient.InstanceId,
                Scoped = _scoped.InstanceId,
                Singleton = _singleton.InstanceId
            },
            ChildService = new
            {
                Transient = _childService.Transient.InstanceId,
                Scoped = _childService.Scoped.InstanceId,
                Singleton = _childService.Singleton.InstanceId
            }
        });
    }
}
```

### Expected Output

**First request:**

```json
{
  "controller": {
    "transient": "a1b2c3d4-...",
    "scoped":    "e5f6a7b8-...",
    "singleton": "11111111-..."
  },
  "childService": {
    "transient": "c9d0e1f2-...",   // DIFFERENT from controller (new instance)
    "scoped":    "e5f6a7b8-...",   // SAME as controller (shared per request)
    "singleton": "11111111-..."    // SAME as controller (shared globally)
  }
}
```

**Second request:**

```json
{
  "controller": {
    "transient": "f3a4b5c6-...",   // DIFFERENT from first request
    "scoped":    "d7e8f9a0-...",   // DIFFERENT from first request (new scope)
    "singleton": "11111111-..."    // SAME as first request (app-wide)
  },
  "childService": {
    "transient": "b1c2d3e4-...",   // DIFFERENT from everything
    "scoped":    "d7e8f9a0-...",   // SAME as controller within this request
    "singleton": "11111111-..."    // SAME as always
  }
}
```

> [!example]
> Key observations from the output:
> - **Transient**: All four Guid values across both requests are different. Every injection creates a new instance.
> - **Scoped**: Within a single request, the controller and child service share the same Guid. Across requests, the Guid changes.
> - **Singleton**: The same Guid appears everywhere, in both requests. It never changes until the app restarts.

> [!summary] Section Summary
> - A service that generates a Guid in its constructor makes lifetime behavior directly observable.
> - Transient: every injection point gets a unique Guid, even within the same request.
> - Scoped: the same Guid is shared across all injection points within one request, but changes per request.
> - Singleton: one Guid for the entire application lifetime, shared everywhere.
> - This pattern is an excellent debugging technique when you suspect lifetime misconfiguration.

---

## The Captive Dependency Problem

The **captive dependency** (also called **captured dependency**) is one of the most insidious DI bugs. It occurs when a longer-lived service holds a reference to a shorter-lived service, causing the shorter-lived service to live far beyond its intended lifetime.

### The Dangerous Code

```csharp
// Registration
builder.Services.AddSingleton<IOrderNotificationService, OrderNotificationService>();
builder.Services.AddScoped<IOrderRepository, OrderRepository>(); // Uses DbContext internally

// The problematic service
public class OrderNotificationService : IOrderNotificationService
{
    private readonly IOrderRepository _orderRepo; // CAPTIVE! Scoped inside a Singleton

    public OrderNotificationService(IOrderRepository orderRepo)
    {
        _orderRepo = orderRepo;
    }

    public async Task NotifyOrderShippedAsync(int orderId)
    {
        // This repository (and its DbContext) was created once when the
        // singleton was first resolved. It is now stale and will be used
        // for ALL future requests -- it will never be disposed or refreshed.
        var order = await _orderRepo.GetByIdAsync(orderId);
        // ... send notification
    }
}
```

### Why This Is Dangerous

1. **Stale DbContext.** The `OrderRepository` (and its `DbContext`) was resolved once when the singleton was created. It holds onto the same database connection and change tracker forever.
2. **Memory leak.** The change tracker accumulates every entity ever loaded, growing without bound.
3. **Connection exhaustion.** The held connection may time out, be reclaimed by the pool, or become stale -- leading to `SqlException` at runtime.
4. **Thread safety violation.** The singleton is accessed by multiple concurrent requests, but the scoped `DbContext` inside it is not thread-safe.
5. **No per-request isolation.** The scoped service was designed to be fresh per request, but as a captive, it is shared across all requests.

> [!danger]
> The captive dependency problem is especially treacherous because it often works correctly in development (single user, low traffic) and only fails under production load or after the application has been running for a while.

### The Fix: Use IServiceScopeFactory

The correct approach is for the singleton to create a new scope each time it needs to access a scoped service:

```csharp
public class OrderNotificationService : IOrderNotificationService
{
    private readonly IServiceScopeFactory _scopeFactory;

    public OrderNotificationService(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }

    public async Task NotifyOrderShippedAsync(int orderId)
    {
        // Create a fresh scope -- gets a fresh DbContext and repository
        using var scope = _scopeFactory.CreateScope();
        var orderRepo = scope.ServiceProvider
            .GetRequiredService<IOrderRepository>();

        var order = await orderRepo.GetByIdAsync(orderId);
        // ... send notification
    }
    // Scope disposed here -- DbContext cleaned up properly
}
```

> [!tip]
> The rule is simple: **services can only depend on services with an equal or longer lifetime.**
> - Singleton can depend on: Singleton
> - Scoped can depend on: Scoped, Singleton
> - Transient can depend on: Transient, Scoped, Singleton
>
> If a singleton needs a scoped service, use `IServiceScopeFactory` to create a scope on demand.

> [!ad-note]
> The same problem applies to a Scoped service holding a Transient dependency. The transient instance is captured for the lifetime of the scope (the full request), which is usually acceptable but may not be if the transient was designed to be truly ephemeral. In practice, the Singleton-captures-Scoped case is by far the most common and dangerous.

See also: [[Common DI Pitfalls]] for more captive dependency scenarios.

> [!summary] Section Summary
> - A captive dependency occurs when a longer-lived service captures a shorter-lived service.
> - The most dangerous case: a Singleton holding a Scoped service (like DbContext).
> - Symptoms include stale data, memory leaks, connection exhaustion, and thread-safety violations.
> - The fix is to inject `IServiceScopeFactory` and create a new scope for each operation.
> - The lifetime rule: services can only depend on equal or longer-lived services.

---

## ValidateScopes and ValidateOnBuild

ASP.NET Core provides two built-in safety nets to catch lifetime misconfigurations early, both enabled by default in the Development environment.

### ValidateScopes

`ValidateScopes` detects captive dependencies at runtime. When enabled, the container throws an `InvalidOperationException` if you try to resolve a scoped service from the root provider (which is what happens when a singleton captures a scoped service).

```csharp
var builder = WebApplication.CreateBuilder(args);

// Enabled by default in Development. To enable explicitly:
builder.Host.UseDefaultServiceProvider(options =>
{
    options.ValidateScopes = true;
});
```

When a captive dependency is detected, you see an error like:

```
System.InvalidOperationException: Cannot resolve scoped service
'IOrderRepository' from root provider. This is caused by a singleton
service 'IOrderNotificationService' depending on a scoped service.
```

> [!caution]
> `ValidateScopes` is disabled in Production by default for performance reasons. This means captive dependency bugs can slip through to production if you do not test thoroughly in Development. Consider enabling it in staging environments as well.

### ValidateOnBuild

`ValidateOnBuild` checks all service registrations at application startup, before any requests are served. It catches issues like:

- Missing registrations (a service depends on an interface that was never registered)
- Open generic registration errors
- Invalid lifetime combinations (captive dependencies)

```csharp
builder.Host.UseDefaultServiceProvider(options =>
{
    options.ValidateScopes = true;
    options.ValidateOnBuild = true;
});
```

A validation failure at startup looks like:

```
System.AggregateException: Some services are not able to be constructed
(Error while validating the service descriptor
'ServiceType: IOrderNotificationService
Lifetime: Singleton
ImplementationType: OrderNotificationService':
Cannot consume scoped service 'IOrderRepository' from singleton
'OrderNotificationService'.)
```

### Recommended Configuration

```csharp
var builder = WebApplication.CreateBuilder(args);

// The default behavior -- both enabled in Development
// This is what CreateBuilder does internally:
if (builder.Environment.IsDevelopment())
{
    builder.Host.UseDefaultServiceProvider(options =>
    {
        options.ValidateScopes = true;
        options.ValidateOnBuild = true;
    });
}

// For maximum safety in all environments (slight startup cost):
builder.Host.UseDefaultServiceProvider(options =>
{
    options.ValidateScopes = builder.Environment.IsDevelopment();
    options.ValidateOnBuild = true; // Always validate at startup
});
```

> [!tip]
> Even though `ValidateScopes` has a runtime performance cost, `ValidateOnBuild` only runs once at startup. There is almost no reason to disable `ValidateOnBuild` in production -- the one-time startup cost is negligible compared to the debugging pain of a missing registration error at 3 AM.

> [!warning] Common Misconception
> "If `ValidateScopes` does not throw, my lifetimes are all correct." Not necessarily. `ValidateScopes` only catches scoped services resolved from the root provider. If you manually create a scope and resolve services from it, `ValidateScopes` does not check those paths. It is a safety net, not a complete verification.

> [!summary] Section Summary
> - `ValidateScopes` detects captive dependencies at runtime by throwing when scoped services are resolved from the root provider.
> - `ValidateOnBuild` validates all registrations at startup, catching missing dependencies before any request is served.
> - Both are enabled by default in Development via `WebApplication.CreateBuilder`.
> - `ValidateOnBuild` has minimal startup cost and is worth enabling in all environments.
> - These validations are safety nets, not complete guarantees -- thorough testing is still essential.

---

## Comprehensive Summary

> [!tip] Complete Summary
> **The Three Lifetimes:**
> - **Transient** (`AddTransient`) -- new instance per injection. Use for stateless, lightweight services like validators and formatters.
> - **Scoped** (`AddScoped`) -- one instance per HTTP request (per scope). Use for DbContext, repositories, Unit of Work, and anything that needs request-level isolation.
> - **Singleton** (`AddSingleton`) -- one instance for the entire application. Use for caches, configuration, and shared infrastructure. Must be thread-safe.
>
> **Key Rules:**
> - A service can only safely depend on services with an **equal or longer lifetime** (Transient < Scoped < Singleton).
> - Violating this rule creates a **captive dependency** -- a scoped service trapped inside a singleton, leading to stale state, memory leaks, and thread-safety bugs.
> - Use `IServiceScopeFactory` when a singleton needs access to scoped services.
> - `DbContext` should always be Scoped (EF Core's default). `HttpClient` should always go through `IHttpClientFactory`.
>
> **Safety Nets:**
> - `ValidateScopes` catches captive dependencies at runtime (enabled in Development by default).
> - `ValidateOnBuild` catches missing registrations at startup (enabled in Development by default).
> - Consider enabling `ValidateOnBuild` in all environments for early failure detection.
>
> **When in Doubt:** Start with **Scoped**. It is the safest default for business services -- it prevents cross-request leaks (unlike Singleton) and avoids unnecessary instance creation (unlike Transient).

---

## Related Topics

- [[DI Overview]] -- fundamentals of dependency injection in ASP.NET Core
- [[Registration Patterns]] -- `Add*` methods, factory registrations, keyed services, open generics
- [[Common DI Pitfalls]] -- anti-patterns including service locator, captive dependencies, and disposal mistakes
- [[Entity Framework Core]] -- DbContext configuration and lifetime management
- [[Background Services]] -- implementing `IHostedService` and `BackgroundService` with proper scope management
- [[Middleware Pipeline]] -- how ASP.NET Core creates per-request scopes automatically
- [[IHttpClientFactory]] -- managing HTTP connections with proper lifetime handling

---

## Further Reading

- [Microsoft Docs: Dependency injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)
- [Microsoft Docs: Service lifetimes](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#service-lifetimes)
- [Microsoft Docs: IHttpClientFactory](https://learn.microsoft.com/en-us/dotnet/core/extensions/httpclient-factory)
- [Microsoft Docs: Background tasks with hosted services](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services)
- [Andrew Lock: The dangers of the captive dependency](https://andrewlock.net/the-dangers-of-the-captive-dependency-in-asp-net-core/)
- [Steve Collins: ASP.NET Core Dependency Injection Best Practices](https://stevetalkscode.co.uk/dependency-injection-best-practices)
