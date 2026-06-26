---
title: Common DI Pitfalls
date: 2026-06-18
tags: [csharp, asp-net-core, dependency-injection, pitfalls]
aliases: [DI Anti-Patterns, Dependency Injection Mistakes, DI Gotchas]
status: complete
---

# Common DI Pitfalls

> [!abstract] Overview
> ASP.NET Core's built-in dependency injection container is powerful but comes with subtle traps that can introduce bugs ranging from stale data to memory leaks to runtime crashes. This note catalogs the most common pitfalls -- captive dependencies, service locator abuse, circular dependencies, thread safety issues, and more -- with concrete code examples showing both the broken code and the correct fix. Understanding these pitfalls is essential before building any non-trivial ASP.NET Core application.

---

## Table of Contents

- [[#Captive Dependency (Lifestyle Mismatch)]]
- [[#Service Locator Anti-Pattern]]
- [[#Constructor Over-Injection]]
- [[#Disposable Service Mismanagement]]
- [[#Resolving Services in Configure or Middleware]]
- [[#Circular Dependencies]]
- [[#Thread Safety with Singletons]]
- [[#Missing Registration Errors]]
- [[#Using new Inside a DI-Managed Service]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]
- [[#Further Reading]]

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

---

## Service Locator Anti-Pattern

> [!info] Definition
> The **Service Locator** anti-pattern occurs when a class takes a dependency on `IServiceProvider` and manually resolves its dependencies using `GetService<T>()` or `GetRequiredService<T>()`, instead of declaring its dependencies explicitly through constructor injection.

### The Anti-Pattern

```csharp
// OrderProcessingService.cs -- SERVICE LOCATOR ANTI-PATTERN
public class OrderProcessingService : IOrderProcessingService
{
    private readonly IServiceProvider _serviceProvider;

    // The only declared dependency is the service locator itself.
    // What does this class ACTUALLY need? You cannot tell from the constructor.
    public OrderProcessingService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public async Task ProcessOrderAsync(Order order)
    {
        // Hidden dependencies -- resolved at runtime, invisible to the consumer
        var inventoryService = _serviceProvider.GetRequiredService<IInventoryService>();
        var paymentGateway = _serviceProvider.GetRequiredService<IPaymentGateway>();
        var emailService = _serviceProvider.GetRequiredService<IEmailService>();
        var auditLogger = _serviceProvider.GetRequiredService<IAuditLogger>();

        await inventoryService.ReserveStockAsync(order.Items);
        await paymentGateway.ChargeAsync(order.Total);
        await emailService.SendOrderConfirmationAsync(order);
        await auditLogger.LogAsync($"Order {order.Id} processed");
    }
}
```

### Why It Is Bad

- **Hidden dependencies**: The constructor signature `(IServiceProvider)` tells you nothing about what the class actually needs. You must read the entire implementation to discover its real dependencies.
- **Testing is painful**: In unit tests, you must set up a mock `IServiceProvider` that returns mocks for every service the class resolves internally. If the implementation adds a new `GetRequiredService<T>()` call, existing tests break with no compiler warning.
- **No compile-time safety**: If `IInventoryService` is not registered, you only find out at runtime when `GetRequiredService` throws -- and only when the specific code path is hit.
- **Defeats the purpose of DI**: The whole point of DI is to make dependencies explicit, visible, and substitutable. Service Locator inverts this back to opaque, hidden resolution.

### The Correct Approach

```csharp
// OrderProcessingService.cs -- PROPER CONSTRUCTOR INJECTION
public class OrderProcessingService : IOrderProcessingService
{
    private readonly IInventoryService _inventoryService;
    private readonly IPaymentGateway _paymentGateway;
    private readonly IEmailService _emailService;
    private readonly IAuditLogger _auditLogger;

    // All dependencies are explicit and visible.
    // You know exactly what this class needs by looking at its constructor.
    public OrderProcessingService(
        IInventoryService inventoryService,
        IPaymentGateway paymentGateway,
        IEmailService emailService,
        IAuditLogger auditLogger)
    {
        _inventoryService = inventoryService;
        _paymentGateway = paymentGateway;
        _emailService = emailService;
        _auditLogger = auditLogger;
    }

    public async Task ProcessOrderAsync(Order order)
    {
        await _inventoryService.ReserveStockAsync(order.Items);
        await _paymentGateway.ChargeAsync(order.Total);
        await _emailService.SendOrderConfirmationAsync(order);
        await _auditLogger.LogAsync($"Order {order.Id} processed");
    }
}
```

### When Service Locator IS Acceptable

There are legitimate scenarios where resolving from `IServiceProvider` is the right choice:

**1. Factory patterns where the type is determined at runtime:**

```csharp
public class NotificationDispatcher
{
    private readonly IServiceProvider _serviceProvider;

    public NotificationDispatcher(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public async Task DispatchAsync(Notification notification)
    {
        // The concrete handler type is not known until runtime.
        // This is a valid use of IServiceProvider.
        var handlerType = typeof(INotificationHandler<>)
            .MakeGenericType(notification.GetType());

        var handler = _serviceProvider.GetRequiredService(handlerType);

        await ((dynamic)handler).HandleAsync(notification);
    }
}
```

**2. Middleware that needs Scoped services:**

```csharp
// In middleware, scoped services must be resolved from HttpContext.RequestServices,
// which is essentially using a service locator -- but this is by design.
public class TenantMiddleware
{
    private readonly RequestDelegate _next;

    public TenantMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Resolving from the request scope -- acceptable in middleware
        var tenantService = context.RequestServices
            .GetRequiredService<ITenantService>();

        tenantService.SetCurrentTenant(context.Request.Headers["X-Tenant-Id"]);
        await _next(context);
    }
}
```

**3. Background services that need to create scopes** (as shown in the [[#Captive Dependency (Lifestyle Mismatch)]] section with `IServiceScopeFactory`).

> [!tip]
> The key distinction: using `IServiceProvider` is acceptable when the **type** to resolve is not known at compile time, or when you are in infrastructure code (middleware, hosted services) that must bridge between DI scopes. It is an anti-pattern when used to hide dependencies that are perfectly well known at compile time.

> [!summary] Section Summary
> - Service Locator hides dependencies behind `IServiceProvider`, making code harder to understand and test
> - Constructor injection makes all dependencies explicit, visible, and verifiable at compile time
> - Service Locator is acceptable for runtime-determined types, middleware scope bridging, and background service scoping
> - If you know the concrete interface you need at compile time, always prefer constructor injection
> - A class taking `IServiceProvider` as its only dependency is a strong code smell

---

## Constructor Over-Injection

> [!info] Definition
> **Constructor over-injection** is a code smell where a class requires an excessive number of dependencies (commonly 7 or more), typically indicating that the class violates the Single Responsibility Principle (SRP).

### The Code Smell

```csharp
// OrderController.cs -- TOO MANY DEPENDENCIES
public class OrderController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly ICustomerService _customerService;
    private readonly IInventoryService _inventoryService;
    private readonly IPaymentGateway _paymentGateway;
    private readonly IShippingCalculator _shippingCalculator;
    private readonly ITaxService _taxService;
    private readonly IDiscountEngine _discountEngine;
    private readonly IEmailService _emailService;
    private readonly IAuditLogger _auditLogger;
    private readonly IConfiguration _configuration;

    public OrderController(
        IOrderService orderService,
        ICustomerService customerService,
        IInventoryService inventoryService,
        IPaymentGateway paymentGateway,
        IShippingCalculator shippingCalculator,
        ITaxService taxService,
        IDiscountEngine discountEngine,
        IEmailService emailService,
        IAuditLogger auditLogger,
        IConfiguration configuration)
    {
        _orderService = orderService;
        _customerService = customerService;
        _inventoryService = inventoryService;
        _paymentGateway = paymentGateway;
        _shippingCalculator = shippingCalculator;
        _taxService = taxService;
        _discountEngine = discountEngine;
        _emailService = emailService;
        _auditLogger = auditLogger;
        _configuration = configuration;
    }

    // This class likely handles order creation, pricing, payment, shipping,
    // notifications, and auditing. That is way too many responsibilities.
}
```

### Solution 1: Introduce a Facade Service

Group related dependencies into a higher-level service that encapsulates a coherent set of operations:

```csharp
// IOrderPricingService.cs -- Groups pricing-related concerns
public interface IOrderPricingService
{
    OrderPricing CalculatePricing(Order order, Customer customer);
}

public class OrderPricingService : IOrderPricingService
{
    private readonly IShippingCalculator _shippingCalculator;
    private readonly ITaxService _taxService;
    private readonly IDiscountEngine _discountEngine;

    public OrderPricingService(
        IShippingCalculator shippingCalculator,
        ITaxService taxService,
        IDiscountEngine discountEngine)
    {
        _shippingCalculator = shippingCalculator;
        _taxService = taxService;
        _discountEngine = discountEngine;
    }

    public OrderPricing CalculatePricing(Order order, Customer customer)
    {
        var discount = _discountEngine.Calculate(order, customer);
        var shipping = _shippingCalculator.Calculate(order);
        var tax = _taxService.Calculate(order, customer.Address);
        return new OrderPricing(discount, shipping, tax);
    }
}
```

```csharp
// OrderController.cs -- AFTER REFACTORING
public class OrderController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly ICustomerService _customerService;
    private readonly IOrderPricingService _pricingService;
    private readonly IPaymentGateway _paymentGateway;
    private readonly IEmailService _emailService;

    public OrderController(
        IOrderService orderService,
        ICustomerService customerService,
        IOrderPricingService pricingService,
        IPaymentGateway paymentGateway,
        IEmailService emailService)
    {
        _orderService = orderService;
        _customerService = customerService;
        _pricingService = pricingService;
        _paymentGateway = paymentGateway;
        _emailService = emailService;
    }
}
```

### Solution 2: Use the Mediator Pattern

With a mediator (such as MediatR), the controller delegates to command/query handlers and needs only the mediator itself:

```csharp
// OrderController.cs -- MEDIATOR APPROACH
public class OrderController : ControllerBase
{
    private readonly IMediator _mediator;

    public OrderController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    public async Task<IActionResult> CreateOrder(CreateOrderRequest request)
    {
        var result = await _mediator.Send(new CreateOrderCommand(request));
        return Ok(result);
    }
}
```

```csharp
// CreateOrderCommandHandler.cs -- Each handler has only the dependencies it needs
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, OrderResult>
{
    private readonly IOrderService _orderService;
    private readonly IPaymentGateway _paymentGateway;

    public CreateOrderCommandHandler(
        IOrderService orderService,
        IPaymentGateway paymentGateway)
    {
        _orderService = orderService;
        _paymentGateway = paymentGateway;
    }

    public async Task<OrderResult> Handle(
        CreateOrderCommand command, CancellationToken cancellationToken)
    {
        // Focused handler with only the dependencies it actually uses
        var order = await _orderService.CreateAsync(command.Request);
        await _paymentGateway.ChargeAsync(order.Total);
        return new OrderResult(order.Id);
    }
}
```

> [!tip]
> There is no magic number, but if your constructor has more than 5-7 parameters, stop and ask: "Is this class doing too many things?" Constructor over-injection is a symptom, not the disease. The disease is SRP violation.

> [!summary] Section Summary
> - A constructor with 7+ dependencies is a code smell indicating the class has too many responsibilities
> - Group related dependencies into focused facade/aggregate services to reduce parameter counts
> - The Mediator pattern (MediatR) moves logic into small, focused handlers with minimal dependencies
> - Always treat constructor over-injection as a signal to refactor the class, not just reduce parameter count
> - Splitting the class into smaller, focused classes is often the best solution

---

## Disposable Service Mismanagement

The DI container manages the lifecycle of services it creates -- including calling `Dispose()` when appropriate. Misunderstanding this leads to either double-disposal bugs or resource leaks.

### Do Not Manually Dispose Injected Services

When the container creates a service, the container is responsible for disposing it. If you dispose it yourself, other consumers of that same instance get an `ObjectDisposedException`.

```csharp
// CustomerService.cs -- DANGEROUS MANUAL DISPOSAL
public class CustomerService : ICustomerService
{
    private readonly AppDbContext _dbContext;

    public CustomerService(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public async Task<Customer> GetCustomerAsync(int id)
    {
        var customer = await _dbContext.Customers.FindAsync(id);

        // DO NOT DO THIS. The container will dispose the DbContext
        // when the scope ends. If you dispose it here, any other service
        // in the same scope that uses this DbContext will crash.
        _dbContext.Dispose(); // BAD -- causes ObjectDisposedException elsewhere
        
        return customer;
    }
}
```

> [!danger] Double Disposal
> If `CustomerService` disposes the `DbContext`, and then `OrderService` (in the same HTTP request scope) tries to use the same `DbContext` instance, it gets `ObjectDisposedException: Cannot access a disposed object`. The container will also try to dispose it again at scope end, causing a double-disposal scenario.

### When YOU Must Handle Disposal

If you create instances yourself (outside the container), the container does not know about them and will not dispose them:

```csharp
// FileExportService.cs -- YOU create it, YOU dispose it
public class FileExportService : IFileExportService
{
    public async Task ExportOrdersAsync(IEnumerable<Order> orders, string filePath)
    {
        // You created this StreamWriter with 'new' -- it is YOUR responsibility
        await using var writer = new StreamWriter(filePath);
        foreach (var order in orders)
        {
            await writer.WriteLineAsync($"{order.Id},{order.Total},{order.Date}");
        }
        // The 'await using' ensures disposal here -- correct
    }
}
```

Similarly, if you register a factory delegate that calls `new`:

```csharp
// Registration with a factory delegate
builder.Services.AddTransient<IReportWriter>(sp =>
{
    // The container calls this factory, but the instance created by 'new'
    // IS tracked by the container because it implements IDisposable.
    // The container WILL dispose it when the scope ends.
    return new CsvReportWriter(sp.GetRequiredService<IConfiguration>());
});
```

> [!ad-note]
> For services registered through the container (even via factory delegates), the container tracks `IDisposable` and `IAsyncDisposable` implementations and disposes them when their scope ends. The exception is Singleton services, which are disposed when the application shuts down (when the root `ServiceProvider` is disposed).

### IDisposable vs IAsyncDisposable

If your service implements `IAsyncDisposable`, the container will call `DisposeAsync()` instead of `Dispose()`. If it implements both, `DisposeAsync()` takes priority:

```csharp
public class OrderRepository : IOrderRepository, IAsyncDisposable
{
    private readonly DbConnection _connection;

    public OrderRepository(DbConnection connection)
    {
        _connection = connection;
    }

    public async ValueTask DisposeAsync()
    {
        // The container calls this automatically when the scope ends
        if (_connection.State == ConnectionState.Open)
        {
            await _connection.CloseAsync();
        }
        await _connection.DisposeAsync();
    }
}
```

> [!summary] Section Summary
> - Never call `Dispose()` on services that were injected by the container -- the container manages their lifecycle
> - The container disposes Scoped services at the end of the HTTP request, Singletons at application shutdown
> - If you create an object with `new` outside the container, you are responsible for disposing it
> - Factory-registered services are still tracked and disposed by the container
> - Prefer `IAsyncDisposable` over `IDisposable` for services with async cleanup (database connections, streams)

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
> Convention-based middleware classes have their **constructor** called once (like a Singleton), but their `InvokeAsync` method is called per request. ASP.NET Core resolves the method parameters from the request scope, making it safe to accept Scoped services as method parameters but NOT as constructor parameters. See [[DI Overview]] for more on middleware DI.

> [!summary] Section Summary
> - `app.Services` is the root `IServiceProvider` -- resolving Scoped services from it creates a root-scoped instance that never disposes
> - For startup/seeding operations, wrap the resolution in `app.Services.CreateScope()`
> - In middleware, use `HttpContext.RequestServices` or accept Scoped services as `InvokeAsync` method parameters
> - Never inject Scoped services into a middleware constructor -- middleware constructors are resolved once like Singletons
> - This is fundamentally the same bug as a captive dependency, just in a different location

---

## Circular Dependencies

> [!info] Definition
> A **circular dependency** occurs when Service A depends on Service B, and Service B depends on Service A (directly or through a chain). The DI container cannot resolve either service because each one requires the other to be constructed first.

### The Problem

```csharp
public class OrderService : IOrderService
{
    private readonly IInventoryService _inventoryService;

    public OrderService(IInventoryService inventoryService)
    {
        _inventoryService = inventoryService;
    }

    public void CreateOrder(Order order)
    {
        _inventoryService.ReserveStock(order.Items);
    }
}

public class InventoryService : IInventoryService
{
    private readonly IOrderService _orderService;

    public InventoryService(IOrderService orderService)
    {
        _orderService = orderService;
    }

    public void ReserveStock(List<OrderItem> items)
    {
        // Needs to check pending orders before reserving...
        var pendingOrders = _orderService.GetPendingOrders();
    }
}
```

### The Error

At runtime, when the container tries to resolve either service, you get a stack overflow or this exception:

```
System.InvalidOperationException: A circular dependency was detected for the service
of type 'IOrderService'.
IOrderService -> IInventoryService -> IOrderService
```

### Fix 1: Restructure to Eliminate the Circle

The best fix is to ask why the circular dependency exists and break it by extracting the shared logic:

```csharp
// Extract the shared concern into its own service
public interface IPendingOrderQuery
{
    List<Order> GetPendingOrders();
}

public class PendingOrderQuery : IPendingOrderQuery
{
    private readonly AppDbContext _dbContext;

    public PendingOrderQuery(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public List<Order> GetPendingOrders()
    {
        return _dbContext.Orders
            .Where(o => o.Status == OrderStatus.Pending)
            .ToList();
    }
}

// Now InventoryService depends on IPendingOrderQuery, not IOrderService
public class InventoryService : IInventoryService
{
    private readonly IPendingOrderQuery _pendingOrderQuery;

    public InventoryService(IPendingOrderQuery pendingOrderQuery)
    {
        _pendingOrderQuery = pendingOrderQuery;
    }

    public void ReserveStock(List<OrderItem> items)
    {
        var pendingOrders = _pendingOrderQuery.GetPendingOrders();
        // ... reservation logic
    }
}
```

### Fix 2: Use Lazy<T> to Break the Cycle

If restructuring is not immediately feasible, `Lazy<T>` defers resolution and breaks the cycle:

```csharp
// Registration
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<IInventoryService, InventoryService>();
builder.Services.AddScoped(sp =>
    new Lazy<IOrderService>(() => sp.GetRequiredService<IOrderService>()));
```

```csharp
public class InventoryService : IInventoryService
{
    private readonly Lazy<IOrderService> _orderService;

    public InventoryService(Lazy<IOrderService> orderService)
    {
        _orderService = orderService;
    }

    public void ReserveStock(List<OrderItem> items)
    {
        // The IOrderService is only resolved when .Value is first accessed,
        // which is AFTER construction -- breaking the circular dependency.
        var pendingOrders = _orderService.Value.GetPendingOrders();
    }
}
```

> [!warning] Common Misconception
> `Lazy<T>` does not fix the underlying design problem -- it only defers it. The circular dependency still exists conceptually, and the code remains tightly coupled. Use `Lazy<T>` as a temporary bridge while you work on a proper restructuring.

### Fix 3: Introduce a Mediating Interface

Create an interface that both services can communicate through without depending on each other:

```csharp
public interface IStockReservationMediator
{
    void ReserveStock(List<OrderItem> items);
    List<Order> GetPendingOrders();
}

public class StockReservationMediator : IStockReservationMediator
{
    private readonly AppDbContext _dbContext;

    public StockReservationMediator(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public void ReserveStock(List<OrderItem> items)
    {
        // Direct database logic -- no dependency on either service
    }

    public List<Order> GetPendingOrders()
    {
        return _dbContext.Orders
            .Where(o => o.Status == OrderStatus.Pending)
            .ToList();
    }
}
```

> [!summary] Section Summary
> - Circular dependencies cause `InvalidOperationException` or stack overflow at resolution time
> - The best fix is restructuring: extract shared logic into a new service that breaks the cycle
> - `Lazy<T>` defers resolution and breaks the cycle technically, but the design problem remains
> - A mediating interface/service can sit between two services to eliminate the direct dependency
> - Circular dependencies are always a design smell -- they indicate responsibilities need to be redistributed

---

## Thread Safety with Singletons

Singleton services are created once and shared across **all** HTTP requests on **all** threads simultaneously. Any mutable state within a Singleton must be explicitly protected against concurrent access.

### The Buggy Singleton

```csharp
// RateLimiter.cs -- NOT THREAD-SAFE
public class RateLimiter : IRateLimiter
{
    // A regular Dictionary is NOT thread-safe for concurrent writes.
    private readonly Dictionary<string, int> _requestCounts = new();
    private DateTime _windowStart = DateTime.UtcNow;

    public bool IsAllowed(string clientIp)
    {
        ResetWindowIfExpired();

        // RACE CONDITION: Two threads for the same IP can both read 0,
        // both increment to 1, and the client gets double the allowed requests.
        if (_requestCounts.ContainsKey(clientIp))
        {
            _requestCounts[clientIp]++;
        }
        else
        {
            _requestCounts[clientIp] = 1; // Can throw on concurrent Add
        }

        return _requestCounts[clientIp] <= 100;
    }

    private void ResetWindowIfExpired()
    {
        if (DateTime.UtcNow - _windowStart > TimeSpan.FromMinutes(1))
        {
            _requestCounts.Clear(); // Can throw if another thread is iterating
            _windowStart = DateTime.UtcNow;
        }
    }
}
```

This code will cause intermittent `InvalidOperationException` ("Collection was modified") crashes under load, plus incorrect counting due to race conditions.

### The Fix with ConcurrentDictionary

```csharp
// RateLimiter.cs -- THREAD-SAFE
public class RateLimiter : IRateLimiter
{
    private readonly ConcurrentDictionary<string, int> _requestCounts = new();
    private DateTime _windowStart = DateTime.UtcNow;
    private readonly object _windowLock = new();

    public bool IsAllowed(string clientIp)
    {
        ResetWindowIfExpired();

        // AddOrUpdate is atomic -- no race condition
        var count = _requestCounts.AddOrUpdate(
            clientIp,
            addValue: 1,
            updateValueFactory: (key, existing) => existing + 1);

        return count <= 100;
    }

    private void ResetWindowIfExpired()
    {
        lock (_windowLock)
        {
            if (DateTime.UtcNow - _windowStart > TimeSpan.FromMinutes(1))
            {
                _requestCounts.Clear();
                _windowStart = DateTime.UtcNow;
            }
        }
    }
}
```

> [!caution] Not Just Dictionaries
> Any mutable state in a Singleton is a potential concurrency bug. This includes:
> - `List<T>`, `Dictionary<TKey, TValue>`, `HashSet<T>` -- use their `Concurrent` counterparts
> - Simple counters (`int _count++`) -- use `Interlocked.Increment`
> - Boolean flags -- use `volatile` or `Interlocked.Exchange`
> - Any shared object being mutated -- protect with `lock` or `SemaphoreSlim` for async code

> [!tip]
> The safest Singleton is a **stateless** Singleton or one with only immutable/read-only state. If your Singleton needs mutable state, consider whether the state should instead live in a Scoped service (per-request), a distributed cache, or a database.

> [!summary] Section Summary
> - Singleton services are shared across all threads simultaneously -- mutable state must be protected
> - Use `ConcurrentDictionary`, `ConcurrentQueue`, etc. instead of their non-thread-safe counterparts
> - Protect compound operations (check-then-act) with `lock` or atomic operations like `Interlocked`
> - The safest approach is to keep Singletons stateless or immutable whenever possible
> - Thread safety bugs in Singletons are intermittent and extremely hard to reproduce in testing

---

## Missing Registration Errors

The most common runtime error when working with DI in ASP.NET Core is forgetting to register a service.

### The Error

```
System.InvalidOperationException: Unable to resolve service for type
'IOrderService' while attempting to activate 'OrderController'.
```

This means the container was asked to create `OrderController`, found that it needs an `IOrderService` in its constructor, and could not find any registration for `IOrderService`.

### Common Causes

**1. Forgot to register the service entirely:**

```csharp
// Program.cs -- Missing registration
builder.Services.AddScoped<ICustomerService, CustomerService>();
// Forgot: builder.Services.AddScoped<IOrderService, OrderService>();

builder.Services.AddControllers();
```

**2. Registered the implementation but not the interface:**

```csharp
// This registers OrderService as itself, NOT as IOrderService
builder.Services.AddScoped<OrderService>();

// The controller asks for IOrderService -- the container cannot match it
public class OrderController : ControllerBase
{
    public OrderController(IOrderService orderService) { } // FAILS
}
```

**3. Interface/implementation mismatch (wrong namespace or assembly):**

```csharp
// Registered IOrderService from one namespace...
builder.Services.AddScoped<Contracts.IOrderService, Services.OrderService>();

// But the controller uses IOrderService from a different namespace
using OldContracts; // Wrong namespace
public class OrderController : ControllerBase
{
    public OrderController(IOrderService orderService) { } // Different IOrderService
}
```

### How to Debug

> [!tip]
> When you see "Unable to resolve service for type X while attempting to activate Y":
> 1. Search `Program.cs` (or your registration code) for the exact interface name
> 2. Verify the registration uses the correct interface (e.g., `AddScoped<IOrderService, OrderService>()`, not just `AddScoped<OrderService>()`)
> 3. Check that the namespace of the interface in the registration matches the namespace used in the consuming class
> 4. If using assembly scanning (e.g., Scrutor), verify the assembly containing the implementation is referenced
> 5. Check for typos -- especially if you have similarly named interfaces like `IOrderService` and `IOrdersService`

### Prevention: ValidateOnBuild

Enable `ValidateOnBuild` to catch missing registrations at application startup rather than at first use:

```csharp
builder.Host.UseDefaultServiceProvider(options =>
{
    options.ValidateOnBuild = true; // Enabled by default in Development
    options.ValidateScopes = true;
});
```

With `ValidateOnBuild = true`, the application fails fast at startup if any registered service has constructor parameters that cannot be resolved, rather than failing only when that specific code path is hit at runtime.

> [!summary] Section Summary
> - "Unable to resolve service" means the container cannot find a registration for the requested type
> - Common causes: forgot to register, registered implementation without interface, namespace mismatch
> - Debug by checking `Program.cs` for the exact interface name, correct generic parameters, and matching namespaces
> - `ValidateOnBuild = true` catches missing registrations at startup instead of at runtime
> - Both `ValidateOnBuild` and `ValidateScopes` are enabled by default in the Development environment

---

## Using `new` Inside a DI-Managed Service

> [!info] Definition
> When a DI-managed service creates its dependencies using `new` instead of receiving them through constructor injection, it defeats the entire purpose of the DI system. The manually created instance does not participate in the DI lifecycle and cannot have its own dependencies injected.

### The Problem

```csharp
// OrderService.cs -- USING new DEFEATS DI
public class OrderService : IOrderService
{
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        // Creating the repository manually with 'new'.
        // But OrderRepository needs a DbContext in its constructor!
        // This either won't compile or requires a parameterless constructor
        // that doesn't use the shared DbContext.
        var repository = new OrderRepository(); // BAD

        // Even if it compiles, the repository is:
        // - Not using the container-managed DbContext
        // - Not participating in the request scope
        // - Not disposable by the container
        // - Impossible to swap out for testing

        var order = new Order(request.CustomerId, request.Items);
        await repository.SaveAsync(order);
        return order;
    }
}
```

```csharp
// The repository expects a DbContext from DI
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _dbContext;

    public OrderRepository(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public async Task SaveAsync(Order order)
    {
        _dbContext.Orders.Add(order);
        await _dbContext.SaveChangesAsync();
    }
}
```

The problems:

- `new OrderRepository()` will not compile because `OrderRepository` requires an `AppDbContext` parameter
- Even if you somehow provide a `DbContext`, it is not the one managed by the container -- transactions, change tracking, and connection pooling all break
- You cannot substitute a mock `IOrderRepository` in unit tests because the dependency is hardcoded
- If `OrderRepository` has its own dependencies (logging, configuration, etc.), those are also missing

### The Fix

```csharp
// OrderService.cs -- PROPER INJECTION
public class OrderService : IOrderService
{
    private readonly IOrderRepository _repository;

    // Let the container provide the repository with all its dependencies wired up
    public OrderService(IOrderRepository repository)
    {
        _repository = repository;
    }

    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        var order = new Order(request.CustomerId, request.Items);
        await _repository.SaveAsync(order);
        return order;
    }
}
```

```csharp
// Registration in Program.cs
builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddScoped<IOrderService, OrderService>();
```

> [!ad-note]
> Using `new` is perfectly fine for data objects, DTOs, value objects, and simple models like `new Order(...)` or `new CreateOrderRequest()`. The rule only applies to **services** -- classes that have behavior and their own dependencies. See [[Registration Patterns]] for guidelines on what should and should not be registered in the container.

> [!summary] Section Summary
> - Using `new` to create service dependencies bypasses the DI container entirely
> - The manually created instance does not get its own dependencies injected (DbContext, loggers, etc.)
> - It creates tight coupling, prevents testing with mocks, and breaks lifecycle management
> - Always inject service dependencies through the constructor and register them in `Program.cs`
> - Using `new` for data objects (DTOs, models, value objects) is perfectly fine -- only service creation is the concern

---

## Comprehensive Summary

> [!tip] Complete Summary
> The most critical DI pitfalls in ASP.NET Core, ranked by frequency and severity:
>
> **Captive Dependency** -- A Singleton captures a Scoped service, causing stale data, memory leaks, and thread-safety violations. Fix with `IServiceScopeFactory`. Enable `ValidateScopes` to catch it early.
>
> **Service Locator** -- Injecting `IServiceProvider` and calling `GetService<T>()` hides dependencies, complicates testing, and removes compile-time safety. Use constructor injection for known types; reserve `IServiceProvider` for runtime-determined types and infrastructure code.
>
> **Constructor Over-Injection** -- 7+ constructor parameters indicates SRP violation. Refactor by introducing facade services, splitting classes, or using the Mediator pattern.
>
> **Disposable Mismanagement** -- Never call `Dispose()` on container-managed services. The container handles disposal for services it creates. You only manage disposal for objects you create with `new`.
>
> **Root Scope Resolution** -- Resolving Scoped services from `app.Services` or middleware constructors creates root-scoped instances. Use `CreateScope()` for startup code and `HttpContext.RequestServices` or method injection for middleware.
>
> **Circular Dependencies** -- A depends on B depends on A causes resolution failure. Fix by restructuring, extracting shared logic, or using `Lazy<T>` as a temporary measure.
>
> **Singleton Thread Safety** -- Singletons are shared across all threads. Protect mutable state with `ConcurrentDictionary`, `lock`, or `Interlocked`. Prefer stateless Singletons.
>
> **Missing Registrations** -- "Unable to resolve service" means a registration is missing, uses the wrong interface, or has a namespace mismatch. Enable `ValidateOnBuild` to catch these at startup.
>
> **Using `new` for Services** -- Manually newing up service dependencies bypasses DI entirely. The created instance gets none of its own dependencies. Always inject through the constructor.
>
> **Golden Rule**: When in doubt, enable both `ValidateScopes` and `ValidateOnBuild` in all environments. The small performance overhead is insignificant compared to the hours spent debugging captive dependencies or missing registrations in production.

---

## Related Topics

- [[DI Overview]] -- Fundamentals of dependency injection in ASP.NET Core
- [[Service Lifetimes]] -- Transient, Scoped, and Singleton lifetime behavior in detail
- [[Registration Patterns]] -- How to register services (AddTransient, AddScoped, AddSingleton, factory delegates, assembly scanning)
- [[IHostedService and Background Services]] -- Background services where captive dependency and scoping pitfalls are most common
- [[Middleware Pipeline]] -- How middleware interacts with DI scopes
- [[Entity Framework Core DbContext]] -- DbContext lifecycle and why it must remain Scoped
- [[Unit Testing with Mocks]] -- Why proper DI matters for testability

---

## Further Reading

- Microsoft Docs: Dependency Injection in ASP.NET Core -- https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- Microsoft Docs: Dependency Injection Guidelines -- https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines
- Mark Seemann, "Dependency Injection Principles, Practices, and Patterns" (Manning, 2nd Edition) -- the definitive book on DI in .NET
- Steve Smith (Ardalis): Avoiding Captive Dependencies -- https://ardalis.com/avoid-captive-dependencies/
- Andrew Lock: Exploring the .NET Core Dependency Injection Container -- https://andrewlock.net/exploring-the-dotnet-core-dependency-injection-container/
