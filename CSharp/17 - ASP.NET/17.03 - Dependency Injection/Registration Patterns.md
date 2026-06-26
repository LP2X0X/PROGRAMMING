---
title: Registration Patterns
date: 2026-06-18
tags: [csharp, asp-net-core, dependency-injection, patterns]
aliases: [DI Registration, Service Registration, DI Patterns]
status: complete
---

# Registration Patterns

> [!abstract] Overview
> ASP.NET Core's built-in dependency injection container supports a wide variety of registration patterns beyond the basic interface-to-concrete mapping. Understanding these patterns -- from factory delegates and open generics to keyed services and assembly scanning -- is essential for writing clean, maintainable `Program.cs` files and building well-structured applications. This note catalogs every major registration pattern with practical examples and guidance on when to reach for each one.

## Table of Contents

- [[#Basic Registration]]
- [[#Self-Registration]]
- [[#Factory Registration]]
- [[#Instance Registration]]
- [[#Multiple Implementations of the Same Interface]]
- [[#TryAdd Methods]]
- [[#Keyed Services (.NET 8+)]]
- [[#Open Generics]]
- [[#Decorator Pattern with DI]]
- [[#Extension Method Pattern]]
- [[#Scanning and Auto-Registration]]
- [[#Registering Options]]
- [[#Real-World Program.cs]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]
- [[#Further Reading]]

---

## Basic Registration

This is the most common registration pattern in ASP.NET Core. You map an interface (the **service type**) to a concrete class (the **implementation type**) with one of three lifetime methods.

> [!info] Definition
> **Basic registration** uses the `Add{Lifetime}<TService, TImplementation>()` extension methods where `TService` is typically an interface and `TImplementation` is the concrete class that implements it.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Transient: new instance every time it is requested
builder.Services.AddTransient<IOrderValidator, OrderValidator>();

// Scoped: one instance per HTTP request (or per scope)
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();

// Singleton: one instance for the entire application lifetime
builder.Services.AddSingleton<ICurrencyConverter, CurrencyConverter>();
```

When the container encounters a constructor parameter of type `IOrderRepository`, it creates (or reuses, depending on the lifetime) an instance of `SqlOrderRepository` and injects it.

```csharp
public class OrderController : ControllerBase
{
    private readonly IOrderRepository _orderRepository;
    private readonly IOrderValidator _validator;

    // Both dependencies resolved automatically by the container
    public OrderController(IOrderRepository orderRepository, IOrderValidator validator)
    {
        _orderRepository = orderRepository;
        _validator = validator;
    }
}
```

> [!tip]
> When in doubt, start with **scoped** for anything that touches a database or per-request state, **singleton** for stateless utilities, and **transient** for lightweight, stateless services that hold no shared state. See [[Service Lifetimes]] for the full decision framework.

> [!summary] Section Summary
> - Basic registration maps an interface to a concrete class using `Add{Lifetime}<TService, TImpl>()`.
> - `AddTransient` creates a new instance on every resolution, `AddScoped` creates one per scope/request, and `AddSingleton` creates one for the app's lifetime.
> - This is the pattern you will use for the vast majority of your registrations.
> - The container automatically resolves constructor parameters by their service type.

---

## Self-Registration

Sometimes a class does not implement an interface -- it **is** the service. In that case, register the concrete type directly.

```csharp
// The service type and implementation type are the same
builder.Services.AddScoped<OrderProcessingService>();
builder.Services.AddSingleton<StartupHealthCheck>();
builder.Services.AddTransient<PdfReportGenerator>();
```

This is equivalent to writing:

```csharp
builder.Services.AddScoped<OrderProcessingService, OrderProcessingService>();
```

> [!ad-note]
> Self-registration is appropriate for **internal implementation details** that are not exposed behind an interface, or for simple utility classes where introducing an interface would add abstraction with no benefit. If the class will ever need to be mocked in tests or swapped for another implementation, prefer registering behind an interface instead.

### When to use self-registration

- **Internal coordinators** that orchestrate other services but are not themselves abstractions.
- **Background services** that derive from `BackgroundService` (though these use `AddHostedService<T>()`).
- **Concrete-only helpers** where testability through mocking is not a concern.

```csharp
public class InventorySyncCoordinator
{
    private readonly IInventoryRepository _inventory;
    private readonly INotificationService _notifications;

    public InventorySyncCoordinator(
        IInventoryRepository inventory,
        INotificationService notifications)
    {
        _inventory = inventory;
        _notifications = notifications;
    }

    public async Task SyncAllWarehousesAsync()
    {
        // Orchestration logic -- no interface needed for this class
    }
}

// Registration
builder.Services.AddScoped<InventorySyncCoordinator>();
```

> [!summary] Section Summary
> - Self-registration uses `Add{Lifetime}<TService>()` with a single type parameter.
> - The service type and implementation type are the same concrete class.
> - Use this for internal services, coordinators, or utilities that do not need an abstraction layer.
> - If the service needs to be mocked or swapped, introduce an interface and use basic registration instead.

---

## Factory Registration

When you need more control over how an instance is created, pass a **factory delegate** to the registration method. The delegate receives an `IServiceProvider` parameter that lets you resolve other services.

```csharp
builder.Services.AddScoped<IOrderService>(sp =>
{
    var repository = sp.GetRequiredService<IOrderRepository>();
    var logger = sp.GetRequiredService<ILogger<OrderService>>();
    var connectionString = sp.GetRequiredService<IConfiguration>()
        .GetConnectionString("Orders");

    return new OrderService(repository, logger, connectionString);
});
```

> [!info] Definition
> **Factory registration** uses the overload `Add{Lifetime}<TService>(Func<IServiceProvider, TService> factory)`. The container calls your delegate each time it needs to create an instance (respecting the chosen lifetime).

### When you need factory registration

**1. Manual constructor parameters that are not services:**

```csharp
builder.Services.AddScoped<IPaymentGateway>(sp =>
{
    var config = sp.GetRequiredService<IConfiguration>();
    var apiKey = config["Payment:ApiKey"];
    var useSandbox = config.GetValue<bool>("Payment:UseSandbox");

    return new StripePaymentGateway(apiKey, useSandbox);
});
```

**2. Conditional logic at creation time:**

```csharp
builder.Services.AddScoped<IShippingCalculator>(sp =>
{
    var config = sp.GetRequiredService<IConfiguration>();
    var provider = config["Shipping:Provider"];

    return provider switch
    {
        "FedEx" => new FedExShippingCalculator(
            sp.GetRequiredService<IHttpClientFactory>()),
        "UPS" => new UpsShippingCalculator(
            sp.GetRequiredService<IHttpClientFactory>()),
        _ => throw new InvalidOperationException(
            $"Unknown shipping provider: {provider}")
    };
});
```

**3. Complex initialization or wrapping:**

```csharp
builder.Services.AddSingleton<ICacheService>(sp =>
{
    var redis = sp.GetRequiredService<IConnectionMultiplexer>();
    var logger = sp.GetRequiredService<ILogger<RedisCacheService>>();
    var cache = new RedisCacheService(redis, logger);

    // Perform initialization that can't happen in the constructor
    cache.WarmUpAsync().GetAwaiter().GetResult();

    return cache;
});
```

> [!warning] Common Misconception
> You might think you need a factory delegate every time a constructor takes a `string` or `int` parameter. Often, a better approach is to use the [[#Registering Options|Options pattern]] and inject `IOptions<T>` into the constructor, keeping the registration clean. Reserve factory delegates for cases where the construction logic is genuinely conditional or complex.

> [!caution]
> Avoid calling `sp.GetRequiredService<T>()` to resolve services with a **shorter lifetime** than the one you are registering. For example, resolving a scoped service inside a singleton factory will capture a single scoped instance forever, leading to the **captive dependency** problem. See [[Common DI Pitfalls]] for details.

> [!summary] Section Summary
> - Factory registration passes a `Func<IServiceProvider, TService>` delegate to the `Add{Lifetime}` method.
> - Use `sp.GetRequiredService<T>()` inside the delegate to resolve other dependencies.
> - Appropriate when you need conditional logic, non-service constructor parameters, or complex initialization.
> - Avoid capturing shorter-lived services inside longer-lived registrations (captive dependency).
> - Consider the Options pattern as an alternative for simple configuration values.

---

## Instance Registration

You can hand a **pre-built object** directly to the container. This is only available for singletons (since the container must return the same instance every time).

```csharp
var smtpSettings = new SmtpSettings
{
    Host = "smtp.company.com",
    Port = 587,
    EnableSsl = true
};

builder.Services.AddSingleton(smtpSettings);

// You can also register it behind an interface
var notifier = new EmailNotificationService(smtpSettings);
builder.Services.AddSingleton<INotificationService>(notifier);
```

> [!danger]
> When you register a pre-built instance, the container does **not** manage its lifecycle. If the object implements `IDisposable`, the container will **not** call `Dispose()` on it when the application shuts down. You are responsible for disposing it yourself (or registering it with a factory delegate instead, which does get disposed).

```csharp
// Container WILL dispose this (factory delegate registration)
builder.Services.AddSingleton<INotificationService>(sp =>
    new EmailNotificationService(sp.GetRequiredService<SmtpSettings>()));

// Container will NOT dispose this (instance registration)
var service = new EmailNotificationService(smtpSettings);
builder.Services.AddSingleton<INotificationService>(service);
```

### When to use instance registration

- **Configuration objects** that you build up manually before the container is constructed.
- **Pre-initialized singletons** that must exist before the DI container is built (rare).
- **Test doubles** in integration tests where you want to inject a specific mock.

> [!summary] Section Summary
> - Instance registration uses `AddSingleton(instance)` or `AddSingleton<TService>(instance)`.
> - Only available for singletons since the same object is returned every time.
> - The container does **not** dispose instance-registered objects -- you must handle disposal yourself.
> - Prefer factory delegate registration if the object implements `IDisposable`.
> - Common for configuration objects and test doubles.

---

## Multiple Implementations of the Same Interface

You can register **multiple concrete types** against the same interface. To consume all of them, inject `IEnumerable<TService>`.

### Registration

```csharp
builder.Services.AddScoped<INotifier, EmailNotifier>();
builder.Services.AddScoped<INotifier, SmsNotifier>();
builder.Services.AddScoped<INotifier, PushNotifier>();
builder.Services.AddScoped<INotifier, SlackNotifier>();
```

### The interface and implementations

```csharp
public interface INotifier
{
    Task SendAsync(string userId, string message);
}

public class EmailNotifier : INotifier
{
    public async Task SendAsync(string userId, string message)
    {
        // Send email notification
    }
}

public class SmsNotifier : INotifier
{
    public async Task SendAsync(string userId, string message)
    {
        // Send SMS notification
    }
}

public class PushNotifier : INotifier
{
    public async Task SendAsync(string userId, string message)
    {
        // Send push notification
    }
}

public class SlackNotifier : INotifier
{
    public async Task SendAsync(string userId, string message)
    {
        // Send Slack message
    }
}
```

### Consuming all implementations

```csharp
public class NotificationDispatcher
{
    private readonly IEnumerable<INotifier> _notifiers;

    public NotificationDispatcher(IEnumerable<INotifier> notifiers)
    {
        _notifiers = notifiers;
    }

    public async Task NotifyAllChannelsAsync(string userId, string message)
    {
        // Iterates over EmailNotifier, SmsNotifier, PushNotifier, SlackNotifier
        foreach (var notifier in _notifiers)
        {
            await notifier.SendAsync(userId, message);
        }
    }
}
```

> [!ad-note]
> If you inject `INotifier` (singular, not `IEnumerable<INotifier>`), you get the **last registered** implementation -- in this case, `SlackNotifier`. This is because each call to `AddScoped<INotifier, T>()` adds a new registration, and the container resolves the last one for singular injection. This behavior is intentional and is how [[#TryAdd Methods]] work.

> [!summary] Section Summary
> - Multiple implementations of the same interface are registered with repeated `Add{Lifetime}` calls.
> - Inject `IEnumerable<TService>` to receive all registered implementations.
> - Injecting the interface directly (not `IEnumerable`) returns the **last registered** implementation.
> - This pattern is ideal for notification systems, validation pipelines, and plugin architectures.
> - With .NET 8+, [[#Keyed Services (.NET 8+)|keyed services]] offer a more targeted alternative when you need a specific implementation rather than all of them.

---

## TryAdd Methods

The `TryAdd` family of methods registers a service **only if** there is no existing registration for that service type. This is critical for library authors.

```csharp
using Microsoft.Extensions.DependencyInjection.Extensions;

// Only registers if IOrderRepository has no existing registration
builder.Services.TryAddScoped<IOrderRepository, SqlOrderRepository>();
builder.Services.TryAddSingleton<ICacheService, MemoryCacheService>();
builder.Services.TryAddTransient<IEmailSender, SmtpEmailSender>();
```

### Why this matters for library authors

Imagine you publish a NuGet package that provides a default `ICacheService`. Without `TryAdd`, your library would **overwrite** whatever the application developer already registered:

```csharp
// === Application's Program.cs ===
builder.Services.AddSingleton<ICacheService, RedisCacheService>(); // App wants Redis

// === Library's extension method (BAD -- overwrites the app's choice) ===
public static IServiceCollection AddMyLibrary(this IServiceCollection services)
{
    services.AddSingleton<ICacheService, MemoryCacheService>(); // Stomps on Redis!
    return services;
}

// === Library's extension method (GOOD -- respects the app's choice) ===
public static IServiceCollection AddMyLibrary(this IServiceCollection services)
{
    services.TryAddSingleton<ICacheService, MemoryCacheService>(); // Skipped, Redis stays
    return services;
}
```

### TryAddEnumerable

There is also `TryAddEnumerable`, which only adds a registration if the **specific implementation type** is not already registered (rather than checking just the service type):

```csharp
// Both get registered because the implementation types differ
services.TryAddEnumerable(ServiceDescriptor.Scoped<INotifier, EmailNotifier>());
services.TryAddEnumerable(ServiceDescriptor.Scoped<INotifier, SmsNotifier>());

// This is skipped because EmailNotifier is already registered for INotifier
services.TryAddEnumerable(ServiceDescriptor.Scoped<INotifier, EmailNotifier>());
```

> [!tip]
> As a rule of thumb: **application code** uses `Add{Lifetime}` (you know exactly what you want). **Library code** uses `TryAdd{Lifetime}` (let the consuming application override your defaults).

> [!summary] Section Summary
> - `TryAdd{Lifetime}` registers a service only if no registration exists for that service type.
> - Essential for library authors to avoid overwriting application-level registrations.
> - `TryAddEnumerable` checks both the service type and the implementation type, preventing duplicate implementations.
> - Requires `using Microsoft.Extensions.DependencyInjection.Extensions;` for the extension methods.
> - Application code typically uses `Add`, library code uses `TryAdd`.

---

## Keyed Services (.NET 8+)

Introduced in .NET 8, **keyed services** let you register multiple implementations of the same interface and resolve a specific one by a string (or object) key. This replaces the older workaround of using factory delegates to pick between named implementations.

### Registration

```csharp
builder.Services.AddKeyedScoped<INotifier, EmailNotifier>("email");
builder.Services.AddKeyedScoped<INotifier, SmsNotifier>("sms");
builder.Services.AddKeyedScoped<INotifier, PushNotifier>("push");
```

### Injection with the `[FromKeyedServices]` attribute

```csharp
public class OrderConfirmationService
{
    private readonly INotifier _emailNotifier;
    private readonly INotifier _smsNotifier;

    public OrderConfirmationService(
        [FromKeyedServices("email")] INotifier emailNotifier,
        [FromKeyedServices("sms")] INotifier smsNotifier)
    {
        _emailNotifier = emailNotifier;
        _smsNotifier = smsNotifier;
    }

    public async Task ConfirmOrderAsync(Order order)
    {
        await _emailNotifier.SendAsync(order.CustomerId, "Order confirmed!");

        if (order.Customer.SmsOptIn)
        {
            await _smsNotifier.SendAsync(order.CustomerId, "Order confirmed!");
        }
    }
}
```

### Resolving keyed services manually

```csharp
public class NotificationRouter
{
    private readonly IServiceProvider _serviceProvider;

    public NotificationRouter(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public async Task SendViaChannelAsync(string channel, string userId, string message)
    {
        var notifier = _serviceProvider.GetRequiredKeyedService<INotifier>(channel);
        await notifier.SendAsync(userId, message);
    }
}
```

> [!ad-note]
> Before .NET 8, selecting a specific implementation from multiple registrations required a factory delegate that switched on a configuration value or parameter. Keyed services formalize this pattern and make it much cleaner. If you are on .NET 8 or later, prefer keyed services over factory-based name resolution.

> [!summary] Section Summary
> - Keyed services associate each registration with a string or object key using `AddKeyed{Lifetime}`.
> - Inject a specific keyed service using the `[FromKeyedServices("key")]` attribute on the constructor parameter.
> - Resolve keyed services manually with `IServiceProvider.GetRequiredKeyedService<T>(key)`.
> - Available in .NET 8 and later; replaces the older factory-delegate pattern for named services.
> - Ideal when you need a specific implementation rather than all implementations via `IEnumerable<T>`.

---

## Open Generics

Open generic registration lets you register a single mapping that covers **all closed generic forms** of an interface.

### The generic interface and implementation

```csharp
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id);
    Task<IReadOnlyList<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

public class SqlRepository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _context;

    public SqlRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<T?> GetByIdAsync(int id)
        => await _context.Set<T>().FindAsync(id);

    public async Task<IReadOnlyList<T>> GetAllAsync()
        => await _context.Set<T>().ToListAsync();

    public async Task AddAsync(T entity)
        => await _context.Set<T>().AddAsync(entity);

    public async Task UpdateAsync(T entity)
        => _context.Set<T>().Update(entity);

    public async Task DeleteAsync(int id)
    {
        var entity = await GetByIdAsync(id);
        if (entity is not null)
            _context.Set<T>().Remove(entity);
    }
}
```

### Registration using `typeof`

```csharp
// One registration covers IRepository<Order>, IRepository<Customer>,
// IRepository<Product>, and every other entity type
builder.Services.AddScoped(typeof(IRepository<>), typeof(SqlRepository<>));
```

### Usage -- the container fills in the type argument automatically

```csharp
public class CustomerController : ControllerBase
{
    private readonly IRepository<Customer> _customers;
    private readonly IRepository<Order> _orders;

    public CustomerController(
        IRepository<Customer> customers,
        IRepository<Order> orders)
    {
        _customers = customers;
        _orders = orders;
    }
}
```

> [!warning] Common Misconception
> You might wonder why you cannot write `builder.Services.AddScoped<IRepository<>, SqlRepository<>>()`. The generic extension method form requires **closed** generic types (e.g., `IRepository<Order>`). Open generics (with empty angle brackets `<>`) are only representable via `typeof()`, so you must use the non-generic `AddScoped(Type, Type)` overload.

> [!ad-note]
> You can combine open generics with specific overrides. Register the open generic first, then register specific closed-generic versions for types that need special handling. The container prefers the specific registration:
>
> ```csharp
> builder.Services.AddScoped(typeof(IRepository<>), typeof(SqlRepository<>));
> builder.Services.AddScoped<IRepository<AuditLog>, ReadOnlyAuditRepository>();
> ```
>
> Now `IRepository<Customer>` resolves to `SqlRepository<Customer>`, but `IRepository<AuditLog>` resolves to `ReadOnlyAuditRepository`.

> [!summary] Section Summary
> - Open generic registration uses `Add{Lifetime}(typeof(IService<>), typeof(Implementation<>))`.
> - A single registration covers every closed form of the generic type.
> - The generic extension method syntax (`Add<T1, T2>`) cannot be used with open generics -- use `typeof`.
> - You can override specific closed-generic types after registering the open generic.
> - This is the standard pattern for generic repository, handler, and pipeline abstractions.

---

## Decorator Pattern with DI

The decorator pattern wraps an existing service with additional behavior (logging, caching, retry logic) without modifying the original implementation.

### The problem

ASP.NET Core's built-in container does not natively support decorators. You cannot simply register two classes for the same interface and have one wrap the other automatically.

### Manual approach with factory registration

```csharp
public interface IOrderService
{
    Task<Order> PlaceOrderAsync(OrderRequest request);
}

public class OrderService : IOrderService
{
    private readonly IOrderRepository _repository;
    private readonly IInventoryService _inventory;

    public OrderService(IOrderRepository repository, IInventoryService inventory)
    {
        _repository = repository;
        _inventory = inventory;
    }

    public async Task<Order> PlaceOrderAsync(OrderRequest request)
    {
        // Core order placement logic
        var order = new Order(request);
        await _inventory.ReserveItemsAsync(order.Items);
        await _repository.SaveAsync(order);
        return order;
    }
}

public class LoggingOrderService : IOrderService
{
    private readonly IOrderService _inner;
    private readonly ILogger<LoggingOrderService> _logger;

    public LoggingOrderService(IOrderService inner, ILogger<LoggingOrderService> logger)
    {
        _inner = inner;
        _logger = logger;
    }

    public async Task<Order> PlaceOrderAsync(OrderRequest request)
    {
        _logger.LogInformation("Placing order for customer {CustomerId}", request.CustomerId);
        var stopwatch = Stopwatch.StartNew();

        var order = await _inner.PlaceOrderAsync(request);

        stopwatch.Stop();
        _logger.LogInformation(
            "Order {OrderId} placed in {ElapsedMs}ms",
            order.Id, stopwatch.ElapsedMilliseconds);

        return order;
    }
}
```

### Registration using a factory delegate

```csharp
builder.Services.AddScoped<OrderService>();
builder.Services.AddScoped<IOrderService>(sp =>
{
    var inner = sp.GetRequiredService<OrderService>();
    var logger = sp.GetRequiredService<ILogger<LoggingOrderService>>();
    return new LoggingOrderService(inner, logger);
});
```

> [!ad-note]
> Notice that `OrderService` is registered as itself (self-registration), while `IOrderService` is registered via a factory that wraps `OrderService` with `LoggingOrderService`. This avoids infinite recursion -- if both were registered as `IOrderService`, resolving `IOrderService` inside the factory would create an infinite loop.

### Stacking multiple decorators

```csharp
builder.Services.AddScoped<OrderService>();

builder.Services.AddScoped<IOrderService>(sp =>
{
    var inner = sp.GetRequiredService<OrderService>();

    // First decorator: logging
    var loggingDecorator = new LoggingOrderService(
        inner,
        sp.GetRequiredService<ILogger<LoggingOrderService>>());

    // Second decorator: caching
    var cachingDecorator = new CachingOrderService(
        loggingDecorator,
        sp.GetRequiredService<IMemoryCache>());

    return cachingDecorator;
});
```

### Using Scrutor for automatic decoration

The third-party library **Scrutor** adds a `Decorate` method that simplifies this pattern significantly:

```csharp
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.Decorate<IOrderService, LoggingOrderService>();
builder.Services.Decorate<IOrderService, CachingOrderService>();
```

> [!tip]
> If you find yourself writing multiple decorator registrations with factory delegates, Scrutor is worth the dependency. It handles the resolution chain correctly and keeps your registration code clean.

> [!summary] Section Summary
> - The built-in DI container does not natively support the decorator pattern.
> - Use a factory delegate: register the inner service as itself, then register the interface with a factory that wraps it.
> - Register the **inner** service by its concrete type to avoid infinite recursion when resolving.
> - Multiple decorators can be stacked by nesting them in the factory delegate.
> - Scrutor (third-party NuGet package) adds `Decorate<TService, TDecorator>()` for clean decorator registration.

---

## Extension Method Pattern

As your `Program.cs` grows, grouping related service registrations into **extension methods** keeps the file organized and readable. This is the standard pattern used by ASP.NET Core itself (e.g., `AddControllers()`, `AddDbContext()`, `AddAuthentication()`).

### Creating a registration extension method

```csharp
public static class OrderProcessingServiceExtensions
{
    public static IServiceCollection AddOrderProcessing(
        this IServiceCollection services)
    {
        services.AddScoped<IOrderService, OrderService>();
        services.AddScoped<IOrderRepository, SqlOrderRepository>();
        services.AddScoped<IOrderValidator, OrderValidator>();
        services.AddScoped<IInventoryService, InventoryService>();
        services.AddTransient<IOrderConfirmationSender, EmailOrderConfirmationSender>();

        return services;
    }
}
```

### Using the extension method in Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOrderProcessing();
builder.Services.AddNotifications();
builder.Services.AddPaymentProcessing();
builder.Services.AddInventoryManagement();

var app = builder.Build();
```

### Extension method with configuration options

```csharp
public static class NotificationServiceExtensions
{
    public static IServiceCollection AddNotifications(
        this IServiceCollection services,
        Action<NotificationOptions>? configure = null)
    {
        if (configure is not null)
        {
            services.Configure(configure);
        }

        services.AddScoped<INotifier, EmailNotifier>();
        services.AddScoped<INotifier, SmsNotifier>();
        services.AddScoped<NotificationDispatcher>();
        services.AddSingleton<INotificationTemplateEngine, RazorNotificationTemplateEngine>();

        return services;
    }
}

// Usage
builder.Services.AddNotifications(options =>
{
    options.DefaultChannel = "email";
    options.RetryCount = 3;
});
```

> [!tip]
> Follow these conventions for registration extension methods:
> - Name the method `Add{Feature}` to match the ASP.NET Core convention.
> - Place it in a `static class` named `{Feature}ServiceExtensions` or `{Feature}ServiceCollectionExtensions`.
> - Always return `IServiceCollection` to enable method chaining.
> - Put the extension class in the `Microsoft.Extensions.DependencyInjection` namespace (for library code) or your application namespace (for app code).

> [!summary] Section Summary
> - Group related registrations into `static` extension methods on `IServiceCollection`.
> - Name them `Add{Feature}()` to follow the ASP.NET Core convention.
> - Always return `IServiceCollection` for method chaining.
> - Optionally accept an `Action<TOptions>` parameter for configurable registration.
> - This pattern keeps `Program.cs` clean and makes registration modules reusable.

---

## Scanning and Auto-Registration

For large applications with many services, manually registering every class becomes tedious. **Assembly scanning** automatically discovers and registers services based on conventions.

### Using Scrutor (recommended third-party approach)

Install the package:

```bash
dotnet add package Scrutor
```

```csharp
builder.Services.Scan(scan => scan
    .FromAssemblyOf<OrderService>()          // Scan the assembly containing OrderService
    .AddClasses(classes => classes
        .AssignableTo<ITransientService>())   // Find all classes implementing ITransientService
    .AsImplementedInterfaces()                // Register them as their interfaces
    .WithTransientLifetime()                  // With transient lifetime

    .AddClasses(classes => classes
        .AssignableTo<IScopedService>())
    .AsImplementedInterfaces()
    .WithScopedLifetime()

    .AddClasses(classes => classes
        .AssignableTo<ISingletonService>())
    .AsImplementedInterfaces()
    .WithSingletonLifetime()
);
```

### Marker interface approach

Define empty marker interfaces to signal the intended lifetime:

```csharp
// Marker interfaces
public interface ITransientService { }
public interface IScopedService { }
public interface ISingletonService { }

// Service implementations declare their lifetime via the marker
public interface IOrderRepository : IScopedService
{
    Task<Order?> GetByIdAsync(int id);
}

public class SqlOrderRepository : IOrderRepository
{
    // Implementation...
}
```

### Manual assembly scanning with reflection

If you prefer not to take a dependency on Scrutor, you can scan assemblies yourself:

```csharp
public static class ServiceRegistrationExtensions
{
    public static IServiceCollection AddServicesFromAssembly(
        this IServiceCollection services,
        Assembly assembly)
    {
        var serviceTypes = assembly.GetTypes()
            .Where(t => t.IsClass && !t.IsAbstract)
            .SelectMany(t => t.GetInterfaces()
                .Where(i => i != typeof(IScopedService)
                          && i != typeof(ITransientService)
                          && i != typeof(ISingletonService))
                .Select(i => new { Interface = i, Implementation = t }));

        foreach (var mapping in serviceTypes)
        {
            if (mapping.Implementation.GetInterfaces().Contains(typeof(IScopedService)))
                services.AddScoped(mapping.Interface, mapping.Implementation);
            else if (mapping.Implementation.GetInterfaces().Contains(typeof(ITransientService)))
                services.AddTransient(mapping.Interface, mapping.Implementation);
            else if (mapping.Implementation.GetInterfaces().Contains(typeof(ISingletonService)))
                services.AddSingleton(mapping.Interface, mapping.Implementation);
        }

        return services;
    }
}

// Usage
builder.Services.AddServicesFromAssembly(typeof(OrderService).Assembly);
```

> [!warning] Common Misconception
> Auto-registration can feel like "magic" -- new services are registered just by implementing an interface, with no visible entry in `Program.cs`. This makes it harder to understand what is registered and can lead to surprises when a class unexpectedly gets picked up by the scanner. For small-to-medium projects, explicit registration (possibly grouped with extension methods) is often clearer.

> [!tip]
> A good middle ground: use Scrutor for the repetitive bulk registrations (repositories, handlers) but still use explicit registration for infrastructure services (database contexts, HTTP clients, caching). This keeps the "important" registrations visible while reducing boilerplate for the "standard" ones.

> [!summary] Section Summary
> - Scrutor provides `Scan()` for convention-based assembly scanning and auto-registration.
> - Marker interfaces (`IScopedService`, `ITransientService`, `ISingletonService`) can signal intended lifetimes.
> - Manual reflection-based scanning is possible but Scrutor handles edge cases better.
> - Auto-registration reduces boilerplate but introduces implicit behavior ("magic").
> - Best used for bulk registrations of similar services; keep infrastructure registrations explicit.

---

## Registering Options

The **Options pattern** is the standard way to bind configuration sections (from `appsettings.json`, environment variables, etc.) to strongly-typed classes and inject them into services.

### 1. Define the settings class

```csharp
public class SmtpSettings
{
    public string Host { get; set; } = string.Empty;
    public int Port { get; set; } = 587;
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
    public bool EnableSsl { get; set; } = true;
    public string FromAddress { get; set; } = string.Empty;
}
```

### 2. Add the configuration section to appsettings.json

```json
{
  "Smtp": {
    "Host": "smtp.company.com",
    "Port": 587,
    "Username": "notifications@company.com",
    "Password": "secret-from-key-vault",
    "EnableSsl": true,
    "FromAddress": "no-reply@company.com"
  }
}
```

### 3. Register the options binding

```csharp
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection("Smtp"));
```

### 4. Inject and use the options

```csharp
public class EmailNotificationService
{
    private readonly SmtpSettings _settings;

    public EmailNotificationService(IOptions<SmtpSettings> options)
    {
        _settings = options.Value;
    }

    public async Task SendAsync(string to, string subject, string body)
    {
        using var client = new SmtpClient(_settings.Host, _settings.Port);
        client.EnableSsl = _settings.EnableSsl;
        client.Credentials = new NetworkCredential(
            _settings.Username, _settings.Password);

        var message = new MailMessage(_settings.FromAddress, to, subject, body);
        await client.SendMailAsync(message);
    }
}
```

### IOptions vs IOptionsSnapshot vs IOptionsMonitor

| Interface | Lifetime | Reloads on Change | Use When |
|---|---|---|---|
| `IOptions<T>` | Singleton | No | Config read once at startup |
| `IOptionsSnapshot<T>` | Scoped | Yes, per request | Config may change; scoped services |
| `IOptionsMonitor<T>` | Singleton | Yes, via `OnChange` callback | Singleton services that need live updates |

```csharp
// For a scoped service that should pick up config changes on each request
public class ReportGenerator
{
    private readonly ReportSettings _settings;

    public ReportGenerator(IOptionsSnapshot<ReportSettings> options)
    {
        _settings = options.Value; // Fresh value each request
    }
}

// For a singleton service that needs to react to config changes
public class BackgroundEmailSender
{
    private SmtpSettings _settings;

    public BackgroundEmailSender(IOptionsMonitor<SmtpSettings> monitor)
    {
        _settings = monitor.CurrentValue;

        monitor.OnChange(newSettings =>
        {
            _settings = newSettings;
        });
    }
}
```

### Options validation

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()  // Validates [Required], [Range], etc.
    .ValidateOnStart();         // Fail fast at startup, not on first use
```

```csharp
public class SmtpSettings
{
    [Required]
    public string Host { get; set; } = string.Empty;

    [Range(1, 65535)]
    public int Port { get; set; } = 587;

    [Required, EmailAddress]
    public string FromAddress { get; set; } = string.Empty;
}
```

> [!tip]
> Always call `.ValidateOnStart()` so that misconfigured settings cause an immediate startup failure rather than a runtime error when the service is first used. This is especially important in production -- you want to fail during deployment, not during a customer request.

> [!summary] Section Summary
> - Use `services.Configure<T>(config.GetSection("..."))` to bind configuration to a strongly-typed class.
> - Inject `IOptions<T>` for static config, `IOptionsSnapshot<T>` for per-request refresh, `IOptionsMonitor<T>` for live singleton updates.
> - Use `AddOptions<T>().BindConfiguration().ValidateDataAnnotations().ValidateOnStart()` for validated, fail-fast configuration.
> - The Options pattern replaces the need for factory delegates that read configuration manually.
> - This is the standard ASP.NET Core approach -- prefer it over injecting `IConfiguration` directly into services.

---

## Real-World Program.cs

Here is a realistic `Program.cs` service registration section that combines the patterns covered above, organized with extension methods and logical grouping.

```csharp
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// -------------------------------------------------------
// Framework services
// -------------------------------------------------------
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// -------------------------------------------------------
// Database
// -------------------------------------------------------
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));

// Generic repository (open generics)
builder.Services.AddScoped(typeof(IRepository<>), typeof(SqlRepository<>));

// -------------------------------------------------------
// Application services (grouped by feature)
// -------------------------------------------------------
builder.Services.AddOrderProcessing();
builder.Services.AddInventoryManagement();
builder.Services.AddNotifications(options =>
{
    options.DefaultChannel = "email";
    options.RetryCount = 3;
});
builder.Services.AddPaymentProcessing();

// -------------------------------------------------------
// Configuration (Options pattern)
// -------------------------------------------------------
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()
    .ValidateOnStart();

builder.Services.AddOptions<PaymentGatewaySettings>()
    .BindConfiguration("PaymentGateway")
    .ValidateDataAnnotations()
    .ValidateOnStart();

// -------------------------------------------------------
// Keyed services (.NET 8+)
// -------------------------------------------------------
builder.Services.AddKeyedScoped<IPaymentProcessor, StripePaymentProcessor>("stripe");
builder.Services.AddKeyedScoped<IPaymentProcessor, PayPalPaymentProcessor>("paypal");
builder.Services.AddKeyedScoped<IPaymentProcessor, SquarePaymentProcessor>("square");

// -------------------------------------------------------
// Infrastructure
// -------------------------------------------------------
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
builder.Services.AddHttpClient<IExternalPricingApi, ExternalPricingApiClient>(client =>
{
    client.BaseAddress = new Uri(
        builder.Configuration["ExternalApis:Pricing:BaseUrl"]!);
    client.Timeout = TimeSpan.FromSeconds(10);
});

// -------------------------------------------------------
// Cross-cutting (decorators)
// -------------------------------------------------------
builder.Services.Decorate<IOrderService, LoggingOrderService>();

// -------------------------------------------------------
// Build and configure pipeline
// -------------------------------------------------------
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### The extension methods behind the scenes

```csharp
public static class OrderProcessingServiceExtensions
{
    public static IServiceCollection AddOrderProcessing(
        this IServiceCollection services)
    {
        services.AddScoped<IOrderService, OrderService>();
        services.AddScoped<IOrderValidator, OrderValidator>();
        services.AddScoped<IOrderPricingEngine, OrderPricingEngine>();
        services.AddTransient<IOrderConfirmationSender, EmailOrderConfirmationSender>();

        return services;
    }
}

public static class InventoryServiceExtensions
{
    public static IServiceCollection AddInventoryManagement(
        this IServiceCollection services)
    {
        services.AddScoped<IInventoryService, InventoryService>();
        services.AddScoped<IWarehouseLocator, WarehouseLocator>();
        services.AddScoped<IStockLevelChecker, StockLevelChecker>();

        return services;
    }
}

public static class PaymentServiceExtensions
{
    public static IServiceCollection AddPaymentProcessing(
        this IServiceCollection services)
    {
        services.AddScoped<IPaymentService, PaymentService>();
        services.AddScoped<IRefundProcessor, RefundProcessor>();
        services.AddScoped<IPaymentAuditLogger, PaymentAuditLogger>();

        return services;
    }
}
```

> [!ad-note]
> Notice the structure: framework services at the top, then database, then application features (via extension methods), then configuration, then infrastructure, then cross-cutting concerns. This ordering is a convention, not a requirement, but it makes `Program.cs` easy to navigate. Each extension method file lives alongside the feature code it registers, not in a central "registration" folder.

> [!summary] Section Summary
> - A well-organized `Program.cs` groups registrations by category with comment separators.
> - Feature-specific registrations are delegated to `Add{Feature}()` extension methods.
> - Framework services, database, options, keyed services, infrastructure, and cross-cutting concerns each get their own section.
> - Extension method classes live alongside the feature code they register.
> - This structure scales well as the application grows -- new features add new extension methods without bloating `Program.cs`.

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Basic registration** (`Add{Lifetime}<TService, TImpl>()`) is the workhorse pattern -- use it for 80% or more of your registrations to map interfaces to concrete types.
>
> **Self-registration** (`Add{Lifetime}<T>()`) registers a concrete class as its own service type, appropriate for internal helpers and coordinators that do not need an abstraction.
>
> **Factory registration** (`Add{Lifetime}<T>(sp => ...)`) gives you full control over construction, needed for conditional logic, manual parameters, or complex initialization. Use `sp.GetRequiredService<T>()` to resolve other dependencies inside the factory.
>
> **Instance registration** (`AddSingleton(instance)`) hands a pre-built object to the container but beware: the container will not dispose it.
>
> **Multiple implementations** of the same interface are consumed via `IEnumerable<T>`, while **keyed services** (.NET 8+) let you resolve a specific implementation by key using `[FromKeyedServices("key")]`.
>
> **TryAdd methods** prevent accidental overwrites and are essential for library authors who need to provide defaults without stomping on application registrations.
>
> **Open generics** (`AddScoped(typeof(IRepo<>), typeof(Repo<>))`) cover all closed forms with one registration, ideal for repositories and handlers.
>
> The **decorator pattern** requires factory delegates (or Scrutor's `Decorate<T1, T2>()`) since the built-in container has no native decorator support.
>
> **Extension methods** (`services.AddFeature()`) are the standard pattern for grouping related registrations and keeping `Program.cs` clean and navigable.
>
> **Assembly scanning** via Scrutor or manual reflection automates bulk registration at the cost of explicitness -- best for large codebases with many similar services.
>
> The **Options pattern** (`Configure<T>`, `IOptions<T>`, `IOptionsSnapshot<T>`, `IOptionsMonitor<T>`) is the standard way to bind configuration to typed classes, with validation and reload support built in.

---

## Related Topics

- [[DI Overview]] -- foundational concepts of dependency injection in ASP.NET Core
- [[Service Lifetimes]] -- transient vs scoped vs singleton in depth
- [[Common DI Pitfalls]] -- captive dependencies, scope validation, and other traps
- [[Options Pattern]] -- deeper dive into `IOptions<T>` and related interfaces
- [[Middleware Pipeline]] -- how DI interacts with the request pipeline
- [[Program.cs Structure]] -- minimal hosting model and application bootstrapping
- [[Unit Testing with DI]] -- mocking and overriding registrations in tests

---

## Further Reading

- [Microsoft Docs: Dependency Injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)
- [Microsoft Docs: Options Pattern in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options)
- [Microsoft Docs: Keyed Services](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection#keyed-services)
- [Scrutor GitHub Repository](https://github.com/khellang/Scrutor)
- [Andrew Lock: Adding Decorated Classes to the ASP.NET Core DI Container](https://andrewlock.net/adding-decorated-classes-to-the-asp-net-core-di-container-using-scrutor/)
- [Steve Smith: ASP.NET Core Dependency Injection Deep Dive](https://ardalis.com/aspnetcore-dependency-injection/)
