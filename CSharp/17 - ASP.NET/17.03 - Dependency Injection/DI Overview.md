---
title: DI Overview
date: 2026-06-18
tags: [csharp, asp-net-core, dependency-injection, fundamentals]
aliases: [Dependency Injection Overview, DI Basics, ASP.NET Core DI]
status: complete
---

# DI Overview

> [!abstract] Overview
> Dependency Injection (DI) is a design pattern in which a class receives its dependencies from an external source rather than creating them internally. ASP.NET Core has a first-class, built-in DI container that manages object creation and lifetime for you. This note covers what DI is, the problems it solves, how the built-in container works, and the main ways to register and resolve services. Understanding DI is foundational to working effectively with ASP.NET Core.

---

## Table of Contents

- [[#What Is Dependency Injection]]
- [[#The Problem DI Solves]]
- [[#Constructor Injection]]
- [[#The Built-in DI Container]]
- [[#How the Container Resolves Dependencies]]
- [[#The builder.Services Property]]
- [[#Registration Approaches]]
- [[#How Controllers Get Injected Dependencies]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]
- [[#Further Reading]]

---

## What Is Dependency Injection

> [!info] Definition
> **Dependency Injection (DI)** is a design pattern where an object receives the other objects it depends on (its *dependencies*) from an external source, rather than creating them itself. The "injection" typically happens through the constructor, though method and property injection also exist.

DI is a specific technique for achieving a broader principle from SOLID: the **Dependency Inversion Principle (DIP)**.

The Dependency Inversion Principle states:

1. **High-level modules should not depend on low-level modules.** Both should depend on abstractions (interfaces or abstract classes).
2. **Abstractions should not depend on details.** Details (concrete implementations) should depend on abstractions.

In practice, this means your `OrderService` should not directly reference a concrete `SqlInventoryRepository`. Instead, it should depend on an `IInventoryRepository` interface. The concrete implementation is decided elsewhere -- typically in the DI container configuration -- and *injected* at runtime.

```csharp
// The high-level module depends on an abstraction, not a concrete class
public class OrderService
{
    private readonly IInventoryRepository _inventory;

    // The dependency is injected -- OrderService does not decide which implementation to use
    public OrderService(IInventoryRepository inventory)
    {
        _inventory = inventory;
    }
}
```

> [!ad-note]
> DI is not unique to ASP.NET Core. It is a general object-oriented design pattern used across languages and frameworks. What ASP.NET Core provides is a *built-in DI container* that automates the process of creating objects and injecting their dependencies. In desktop .NET development, you may have used DI manually or through third-party containers like Autofac or Ninject. ASP.NET Core makes DI a first-class citizen that you interact with on every project.

> [!summary] Section Summary
> - Dependency Injection is a pattern where dependencies are provided to a class from the outside, not created internally.
> - It implements the Dependency Inversion Principle: depend on abstractions (interfaces), not concrete implementations.
> - High-level business logic classes should never directly instantiate their low-level dependencies.
> - ASP.NET Core ships with a built-in DI container, making DI a core part of the framework rather than an optional add-on.

---

## The Problem DI Solves

Without DI, classes create their own dependencies using `new`. This creates **tight coupling** -- the class is permanently bound to specific concrete implementations, making it difficult to test, extend, or change behavior.

### Before: Tight Coupling Without DI

```csharp
public class OrderService
{
    private readonly SqlInventoryRepository _inventory;
    private readonly StripePaymentGateway _paymentGateway;
    private readonly SmtpEmailService _emailService;

    public OrderService()
    {
        // This class decides exactly which implementations to use
        _inventory = new SqlInventoryRepository("Server=prod;Database=Inventory;...");
        _paymentGateway = new StripePaymentGateway("sk_live_abc123");
        _emailService = new SmtpEmailService("smtp.company.com", 587);
    }

    public OrderResult PlaceOrder(Order order)
    {
        bool inStock = _inventory.CheckStock(order.ProductId, order.Quantity);
        if (!inStock)
            return OrderResult.OutOfStock;

        PaymentResult payment = _paymentGateway.Charge(order.Total, order.PaymentInfo);
        if (!payment.Success)
            return OrderResult.PaymentFailed;

        _inventory.ReserveStock(order.ProductId, order.Quantity);
        _emailService.SendConfirmation(order.CustomerEmail, order);

        return OrderResult.Success;
    }
}
```

> [!danger] Problems With This Approach
> - **Untestable**: You cannot unit test `OrderService` without hitting a real database, Stripe API, and SMTP server.
> - **Rigid**: Switching from Stripe to PayPal requires modifying `OrderService` source code.
> - **Hidden dependencies**: Anyone reading the class signature has no idea what it needs to function.
> - **Connection strings and secrets are hardcoded** inside the business logic class.
> - **Violation of Single Responsibility**: `OrderService` is responsible for both its business logic and for deciding which implementations to use.

### After: Loose Coupling With DI

```csharp
public class OrderService : IOrderService
{
    private readonly IInventoryRepository _inventory;
    private readonly IPaymentGateway _paymentGateway;
    private readonly IEmailService _emailService;

    // Dependencies are declared as interfaces and injected through the constructor
    public OrderService(
        IInventoryRepository inventory,
        IPaymentGateway paymentGateway,
        IEmailService emailService)
    {
        _inventory = inventory;
        _paymentGateway = paymentGateway;
        _emailService = emailService;
    }

    public OrderResult PlaceOrder(Order order)
    {
        bool inStock = _inventory.CheckStock(order.ProductId, order.Quantity);
        if (!inStock)
            return OrderResult.OutOfStock;

        PaymentResult payment = _paymentGateway.Charge(order.Total, order.PaymentInfo);
        if (!payment.Success)
            return OrderResult.PaymentFailed;

        _inventory.ReserveStock(order.ProductId, order.Quantity);
        _emailService.SendConfirmation(order.CustomerEmail, order);

        return OrderResult.Success;
    }
}
```

> [!tip] What Changed
> The business logic in `PlaceOrder` is **identical**. The only difference is how `OrderService` gets its dependencies. Now you can:
> - **Unit test** by passing in mock implementations of `IInventoryRepository`, `IPaymentGateway`, and `IEmailService`.
> - **Swap implementations** (e.g., `StripePaymentGateway` to `PayPalPaymentGateway`) without touching `OrderService`.
> - **See all dependencies** just by reading the constructor signature.

> [!example] Unit Testing With DI
> ```csharp
> [Fact]
> public void PlaceOrder_WhenOutOfStock_ReturnsOutOfStock()
> {
>     // Arrange -- using mock implementations
>     var mockInventory = new Mock<IInventoryRepository>();
>     mockInventory
>         .Setup(i => i.CheckStock(It.IsAny<int>(), It.IsAny<int>()))
>         .Returns(false);
> 
>     var mockPayment = new Mock<IPaymentGateway>();
>     var mockEmail = new Mock<IEmailService>();
> 
>     var service = new OrderService(
>         mockInventory.Object,
>         mockPayment.Object,
>         mockEmail.Object);
> 
>     // Act
>     var result = service.PlaceOrder(new Order { ProductId = 1, Quantity = 100 });
> 
>     // Assert
>     Assert.Equal(OrderResult.OutOfStock, result);
>     mockPayment.Verify(p => p.Charge(It.IsAny<decimal>(), It.IsAny<PaymentInfo>()), Times.Never);
> }
> ```

> [!summary] Section Summary
> - Without DI, classes use `new` to create dependencies, leading to tight coupling between high-level and low-level modules.
> - Tightly coupled code is difficult to test because you cannot substitute real services with mocks or fakes.
> - DI solves this by depending on interfaces and receiving concrete implementations from the outside.
> - The business logic itself does not change -- only how dependencies are obtained changes.
> - With DI, unit testing becomes straightforward because you can inject mock implementations.

---

## Constructor Injection

Constructor injection is the **primary and recommended** DI pattern in ASP.NET Core. The idea is simple: a class declares its dependencies as constructor parameters, and the DI container provides them automatically when it creates an instance.

```csharp
public class CustomerController : ControllerBase
{
    private readonly ICustomerService _customerService;
    private readonly ILogger<CustomerController> _logger;

    // Constructor injection -- the framework provides these automatically
    public CustomerController(
        ICustomerService customerService,
        ILogger<CustomerController> logger)
    {
        _customerService = customerService;
        _logger = logger;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetCustomer(int id)
    {
        _logger.LogInformation("Fetching customer {CustomerId}", id);
        var customer = await _customerService.GetByIdAsync(id);

        if (customer is null)
            return NotFound();

        return Ok(customer);
    }
}
```

### Why Constructor Injection Is Preferred

| Advantage | Explanation |
|---|---|
| **Explicit dependencies** | Every dependency is visible in the constructor signature |
| **Immutability** | Dependencies are assigned to `readonly` fields, preventing reassignment |
| **Required by default** | If a required service is not registered, the app fails at startup with a clear error, not at runtime |
| **Framework support** | ASP.NET Core's built-in container resolves constructor parameters automatically |

> [!warning] Common Misconception
> You might think you need to "wire up" each controller manually, passing in the correct service instances. You do not. The DI container reads the constructor parameters, looks up the registered services, creates the necessary instances, and passes them in. You only need to register the services once at startup.

> [!ad-note]
> ASP.NET Core also supports **method injection** via the `[FromServices]` attribute on action method parameters, and **service location** via `HttpContext.RequestServices.GetService<T>()`. However, constructor injection should be your default choice. Method injection is useful for dependencies needed by only one action method. Service location (the Service Locator pattern) is generally considered an anti-pattern because it hides dependencies.

> [!summary] Section Summary
> - Constructor injection is the primary DI pattern in ASP.NET Core -- declare dependencies as constructor parameters.
> - Dependencies should be stored in `private readonly` fields for immutability.
> - The DI container resolves all constructor parameters automatically; no manual wiring is needed.
> - Constructor injection makes dependencies explicit, required, and visible.
> - Prefer constructor injection over method injection or the Service Locator pattern.

---

## The Built-in DI Container

ASP.NET Core ships with a built-in, lightweight DI container. Two key interfaces make it work:

### IServiceCollection -- The Registration Side

`IServiceCollection` is a collection of `ServiceDescriptor` objects. Each descriptor tells the container: "When someone asks for *this type*, give them *that implementation* with *this lifetime*."

You interact with `IServiceCollection` at startup to register all your services.

```csharp
// IServiceCollection is what you add registrations to
IServiceCollection services = builder.Services;

services.AddScoped<IOrderService, OrderService>();
services.AddScoped<IInventoryRepository, SqlInventoryRepository>();
services.AddSingleton<IPaymentGateway, StripePaymentGateway>();
services.AddTransient<IEmailService, SmtpEmailService>();
```

### IServiceProvider -- The Resolution Side

`IServiceProvider` is the actual container that resolves services. Once you have finished registering services and the application builds, the `IServiceCollection` is compiled into an `IServiceProvider`. This is the object that knows how to create instances and manage their lifetimes.

```csharp
// You rarely interact with IServiceProvider directly.
// The framework uses it behind the scenes to resolve dependencies.

// But if you ever need to resolve manually (e.g., in middleware):
public class CustomMiddleware
{
    private readonly RequestDelegate _next;

    public CustomMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Resolving a service from the request's service provider
        var orderService = context.RequestServices.GetRequiredService<IOrderService>();

        // ... use orderService ...

        await _next(context);
    }
}
```

> [!info] Definition
> **IServiceCollection** = the *registration* side. You tell it what services exist and how they map to implementations.
> **IServiceProvider** = the *resolution* side. It creates instances and injects them where needed.

> [!caution]
> Do not hold onto or cache the `IServiceProvider` in your own code and use it to resolve services throughout the application. This is the **Service Locator anti-pattern**. It hides dependencies and makes code harder to test. Let the framework inject dependencies through constructors instead.

> [!summary] Section Summary
> - The built-in DI container consists of two key interfaces: `IServiceCollection` (registration) and `IServiceProvider` (resolution).
> - `IServiceCollection` holds `ServiceDescriptor` entries that map service types to implementations and lifetimes.
> - `IServiceProvider` is built from the collection and knows how to create and manage service instances.
> - You register services into `IServiceCollection` at startup; the framework builds an `IServiceProvider` from it.
> - Avoid using `IServiceProvider` directly in application code -- let the framework inject dependencies via constructors.

---

## How the Container Resolves Dependencies

Understanding the full lifecycle from registration to resolution clarifies what happens "behind the scenes" when ASP.NET Core creates a controller or service.

### Step 1: Registration at Startup

In `Program.cs`, you register services into the `IServiceCollection`:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Registration phase
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<IInventoryRepository, SqlInventoryRepository>();
builder.Services.AddScoped<IPaymentGateway, StripePaymentGateway>();
builder.Services.AddTransient<IEmailService, SmtpEmailService>();
builder.Services.AddControllers();

var app = builder.Build(); // <-- IServiceProvider is created here
```

### Step 2: Building the Provider

When `builder.Build()` is called, the framework compiles all `ServiceDescriptor` entries into an `IServiceProvider`. After this point, no new registrations can be added.

### Step 3: A Request Arrives

When an HTTP request comes in, ASP.NET Core creates a **scope** for that request. This scope has its own `IServiceProvider` that tracks scoped and transient services for the duration of the request.

### Step 4: Controller Activation

The framework determines which controller handles the request. It inspects the controller's constructor to find what parameters are needed:

```csharp
public class OrderController : ControllerBase
{
    // The framework sees: "I need an IOrderService and an ILogger<OrderController>"
    public OrderController(IOrderService orderService, ILogger<OrderController> logger)
    {
        // ...
    }
}
```

### Step 5: Recursive Resolution

The container resolves `IOrderService`, which maps to `OrderService`. But `OrderService` itself has dependencies:

```csharp
public class OrderService : IOrderService
{
    public OrderService(
        IInventoryRepository inventory,    // needs resolving
        IPaymentGateway paymentGateway,    // needs resolving
        IEmailService emailService)        // needs resolving
    {
        // ...
    }
}
```

The container resolves each of these recursively, building the entire dependency graph automatically.

### Step 6: Instance Delivery

The fully constructed `OrderController` (with its `OrderService`, which has its `SqlInventoryRepository`, `StripePaymentGateway`, and `SmtpEmailService`) is handed to the framework to process the request.

> [!ad-note]
> This entire resolution process is automatic. You never write code like `new OrderController(new OrderService(new SqlInventoryRepository(...), ...))`. The container does all of this based on the registrations you provided at startup.

> [!summary] Section Summary
> - Services are registered into `IServiceCollection` during the startup/configuration phase in `Program.cs`.
> - `builder.Build()` compiles registrations into an `IServiceProvider` -- no further registrations after this point.
> - Each HTTP request gets its own scope with a scoped `IServiceProvider`.
> - The container inspects constructor parameters and recursively resolves the entire dependency graph.
> - The result is a fully constructed object tree delivered to the framework without any manual instantiation code.

---

## The builder.Services Property

In modern ASP.NET Core (using the minimal hosting model introduced in .NET 6), all service registration happens through `builder.Services` in `Program.cs`.

```csharp
var builder = WebApplication.CreateBuilder(args);

// builder.Services is an IServiceCollection
// All registrations go here, before builder.Build()

// Framework services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Application services
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<ICustomerService, CustomerService>();
builder.Services.AddScoped<IInventoryRepository, SqlInventoryRepository>();
builder.Services.AddSingleton<IPaymentGateway, StripePaymentGateway>();

// Third-party and infrastructure services
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// After Build(), you configure the middleware pipeline
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

> [!tip] Organizing Registrations
> As your application grows, `Program.cs` can become cluttered with service registrations. A common pattern is to use **extension methods** to group related registrations:
>
> ```csharp
> // In a separate file: ServiceCollectionExtensions.cs
> public static class ServiceCollectionExtensions
> {
>     public static IServiceCollection AddOrderingServices(this IServiceCollection services)
>     {
>         services.AddScoped<IOrderService, OrderService>();
>         services.AddScoped<IInventoryRepository, SqlInventoryRepository>();
>         services.AddScoped<IPaymentGateway, StripePaymentGateway>();
>         services.AddTransient<IEmailService, SmtpEmailService>();
>         return services;
>     }
>
>     public static IServiceCollection AddCustomerServices(this IServiceCollection services)
>     {
>         services.AddScoped<ICustomerService, CustomerService>();
>         services.AddScoped<ICustomerRepository, SqlCustomerRepository>();
>         return services;
>     }
> }
>
> // In Program.cs -- clean and readable
> builder.Services.AddOrderingServices();
> builder.Services.AddCustomerServices();
> ```

> [!warning] Common Misconception
> You might wonder if the order of registrations matters. For most cases, it does not -- the container resolves dependencies based on types, not registration order. However, if you register the **same interface twice**, the last registration wins (the container returns the last-registered implementation). If you need multiple implementations of the same interface, you can inject `IEnumerable<IMyService>` to get all of them.

> [!summary] Section Summary
> - `builder.Services` is the `IServiceCollection` where all service registrations happen in `Program.cs`.
> - Both framework services (`AddControllers`, `AddDbContext`) and your application services are registered here.
> - All registrations must happen before `builder.Build()` is called.
> - Use extension methods on `IServiceCollection` to keep `Program.cs` clean as your application grows.
> - If the same interface is registered multiple times, the last registration wins for single-instance resolution.

---

## Registration Approaches

There are three main ways to register services with the DI container. Each serves a different purpose.

### Interface-Based Registration

The most common and recommended approach. You map an interface (the service type) to a concrete class (the implementation type).

```csharp
// "When someone asks for IOrderService, give them an OrderService"
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<IInventoryRepository, SqlInventoryRepository>();
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
```

This is the approach that enables the Dependency Inversion Principle -- consuming classes depend only on the interface, never on the concrete type.

> [!tip] When to Use Interface-Based Registration
> Use this as your default approach. It allows you to:
> - Swap implementations without changing consuming code
> - Mock dependencies in unit tests
> - Follow SOLID principles

### Self-Registration (Concrete Type Only)

You can register a concrete class without an interface. The service type and implementation type are the same.

```csharp
// "When someone asks for OrderService (the concrete class), create one"
builder.Services.AddScoped<OrderService>();

// Equivalent to:
builder.Services.AddScoped<OrderService, OrderService>();
```

```csharp
// Usage -- the controller depends on the concrete type directly
public class OrderController : ControllerBase
{
    private readonly OrderService _orderService;

    public OrderController(OrderService orderService)
    {
        _orderService = orderService;
    }
}
```

> [!ad-note]
> Self-registration is useful for classes that are unlikely to have multiple implementations, such as configuration objects, helper classes, or application-specific services with no foreseeable need for substitution. However, it does make unit testing harder since you cannot easily substitute a mock without an interface.

### Factory-Based Registration

Factory registration gives you full control over how an instance is created. You provide a lambda that receives the `IServiceProvider` and returns the service instance.

```csharp
// Simple factory
builder.Services.AddScoped<IOrderService>(serviceProvider =>
{
    var inventory = serviceProvider.GetRequiredService<IInventoryRepository>();
    var payment = serviceProvider.GetRequiredService<IPaymentGateway>();
    var email = serviceProvider.GetRequiredService<IEmailService>();
    var logger = serviceProvider.GetRequiredService<ILogger<OrderService>>();

    return new OrderService(inventory, payment, email, logger);
});
```

```csharp
// Factory with conditional logic
builder.Services.AddScoped<IPaymentGateway>(serviceProvider =>
{
    var config = serviceProvider.GetRequiredService<IConfiguration>();
    var environment = config["PaymentEnvironment"];

    return environment switch
    {
        "production" => new StripePaymentGateway(config["Stripe:LiveKey"]!),
        "sandbox" => new StripePaymentGateway(config["Stripe:TestKey"]!),
        _ => new FakePaymentGateway() // For local development
    };
});
```

```csharp
// Factory for classes that need special initialization
builder.Services.AddSingleton<IInventoryCache>(serviceProvider =>
{
    var logger = serviceProvider.GetRequiredService<ILogger<InventoryCache>>();
    var cache = new InventoryCache(logger);
    cache.Preload(); // Custom initialization step
    return cache;
});
```

> [!tip] When to Use Factory Registration
> Use factories when:
> - The constructor needs values that are not themselves registered services (e.g., configuration strings, computed values)
> - You need conditional logic to decide which implementation to return
> - The object requires custom initialization steps beyond what the constructor does
> - You are integrating with third-party classes whose constructors you do not control

### Comparison Table

| Approach | Syntax | Best For |
|---|---|---|
| Interface-based | `AddScoped<IService, Implementation>()` | Most services -- enables substitution and testing |
| Self-registration | `AddScoped<ConcreteClass>()` | Simple classes with no need for abstraction |
| Factory-based | `AddScoped<IService>(sp => ...)` | Complex creation logic, conditional implementations |

> [!summary] Section Summary
> - **Interface-based** (`AddScoped<IService, Implementation>()`) is the default and recommended approach for most services.
> - **Self-registration** (`AddScoped<ConcreteClass>()`) registers a concrete type directly, useful for classes that will not be substituted.
> - **Factory-based** (`AddScoped<IService>(sp => ...)`) provides a lambda for custom construction logic, conditional implementations, or special initialization.
> - Interface-based registration best supports SOLID principles, testability, and implementation swapping.
> - Factory registration is the escape hatch for scenarios where simple type mapping is not enough.

---

## How Controllers Get Injected Dependencies

ASP.NET Core controllers are created by the framework's controller activator, which uses the DI container to resolve all constructor dependencies automatically.

### Complete Example: Registration to Resolution

**Step 1 -- Define the interfaces and implementations:**

```csharp
// Interfaces
public interface IProductService
{
    Task<Product?> GetByIdAsync(int id);
    Task<IEnumerable<Product>> SearchAsync(string query);
    Task<Product> CreateAsync(CreateProductRequest request);
}

public interface IProductRepository
{
    Task<Product?> FindByIdAsync(int id);
    Task<IEnumerable<Product>> SearchAsync(string query);
    Task<Product> AddAsync(Product product);
}

// Implementation
public class ProductService : IProductService
{
    private readonly IProductRepository _repository;
    private readonly ILogger<ProductService> _logger;

    public ProductService(IProductRepository repository, ILogger<ProductService> logger)
    {
        _repository = repository;
        _logger = logger;
    }

    public async Task<Product?> GetByIdAsync(int id)
    {
        _logger.LogInformation("Fetching product {ProductId}", id);
        return await _repository.FindByIdAsync(id);
    }

    public async Task<IEnumerable<Product>> SearchAsync(string query)
    {
        _logger.LogInformation("Searching products with query: {Query}", query);
        return await _repository.SearchAsync(query);
    }

    public async Task<Product> CreateAsync(CreateProductRequest request)
    {
        var product = new Product
        {
            Name = request.Name,
            Price = request.Price,
            Category = request.Category
        };

        _logger.LogInformation("Creating product: {ProductName}", product.Name);
        return await _repository.AddAsync(product);
    }
}
```

**Step 2 -- Register services in Program.cs:**

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Register application services
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddScoped<IProductRepository, SqlProductRepository>();

var app = builder.Build();

app.MapControllers();
app.Run();
```

**Step 3 -- The controller receives everything automatically:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductController : ControllerBase
{
    private readonly IProductService _productService;
    private readonly ILogger<ProductController> _logger;

    // The DI container provides both IProductService and ILogger automatically
    public ProductController(
        IProductService productService,
        ILogger<ProductController> logger)
    {
        _productService = productService;
        _logger = logger;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetProduct(int id)
    {
        var product = await _productService.GetByIdAsync(id);
        if (product is null)
        {
            _logger.LogWarning("Product {ProductId} not found", id);
            return NotFound();
        }
        return Ok(product);
    }

    [HttpGet("search")]
    public async Task<IActionResult> Search([FromQuery] string query)
    {
        var results = await _productService.SearchAsync(query);
        return Ok(results);
    }

    [HttpPost]
    public async Task<IActionResult> CreateProduct([FromBody] CreateProductRequest request)
    {
        var product = await _productService.CreateAsync(request);
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }
}
```

> [!ad-note]
> Notice that `ILogger<T>` is never explicitly registered by you. ASP.NET Core registers logging services automatically as part of `WebApplication.CreateBuilder()`. Many framework services -- logging, configuration, hosting -- are pre-registered for you. You only need to register your own application-specific services.

### What Happens at Runtime

When a request hits `GET /api/product/42`:

1. The framework routes the request to `ProductController.GetProduct`.
2. The controller activator asks the DI container: "Create a `ProductController`."
3. The container sees the constructor needs `IProductService` and `ILogger<ProductController>`.
4. It resolves `IProductService` -> `ProductService`, which needs `IProductRepository` and `ILogger<ProductService>`.
5. It resolves `IProductRepository` -> `SqlProductRepository`.
6. It resolves both `ILogger<>` instances from the pre-registered logging services.
7. It builds the full object graph: `SqlProductRepository` -> `ProductService` -> `ProductController`.
8. The controller's `GetProduct` method executes with all dependencies in place.

> [!warning] Common Misconception
> You do not need to register controllers themselves as services for constructor injection to work. `AddControllers()` handles controller discovery and activation. Controllers are not resolved from the DI container by default -- they are created by the `DefaultControllerActivator`, which uses the container only to resolve the constructor *parameters*. If you want controllers themselves to be container-managed (for filters, etc.), you can call `builder.Services.AddControllers().AddControllersAsServices()`, but this is not required for basic DI.

> [!summary] Section Summary
> - Controllers declare their dependencies as constructor parameters; the framework resolves them automatically.
> - You register your application services in `Program.cs`; framework services like `ILogger<T>` are pre-registered.
> - The DI container builds the entire dependency graph recursively at the time of controller activation.
> - `AddControllers()` handles controller discovery -- you do not register controllers as services for basic DI.
> - The full resolution chain (controller -> service -> repository) happens transparently on every request.

---

## Comprehensive Summary

> [!tip] Complete Summary
> **What DI is:** A design pattern where classes receive dependencies from the outside rather than creating them with `new`. It implements the Dependency Inversion Principle -- depend on abstractions, not concretions.
>
> **Why it matters:** Without DI, code is tightly coupled, difficult to test, and resistant to change. With DI, you can swap implementations, mock dependencies in tests, and see all dependencies at a glance in the constructor.
>
> **How ASP.NET Core implements it:** The framework provides a built-in DI container with two key interfaces -- `IServiceCollection` for registration and `IServiceProvider` for resolution. You register services in `Program.cs` via `builder.Services`, and the container automatically resolves the full dependency graph when creating controllers and other framework-managed objects.
>
> **Three registration approaches:** Interface-based (`AddScoped<IService, Impl>()`) for most services, self-registration (`AddScoped<ConcreteClass>()`) for simple cases, and factory-based (`AddScoped<IService>(sp => ...)`) for complex construction logic.
>
> **Constructor injection** is the primary pattern -- declare dependencies as constructor parameters, store them in `readonly` fields, and let the framework handle the rest. The next topics to explore are [[Service Lifetimes]] (Scoped, Singleton, Transient), [[Registration Patterns]], and [[Common DI Pitfalls]].

---

## Related Topics

- [[Service Lifetimes]] -- Scoped vs Singleton vs Transient and when to use each
- [[Registration Patterns]] -- Advanced registration techniques including keyed services, open generics, and decorator patterns
- [[Common DI Pitfalls]] -- Captive dependencies, service locator anti-pattern, and other mistakes to avoid
- [[Middleware in ASP.NET Core]] -- How middleware interacts with DI
- [[Options Pattern]] -- Using `IOptions<T>` for strongly-typed configuration
- [[Unit Testing with DI]] -- Mocking strategies and test service providers

---

## Further Reading

- [Microsoft Docs: Dependency Injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)
- [Microsoft Docs: Dependency Inversion Principle](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles#dependency-inversion)
- [Microsoft Docs: .NET Dependency Injection Guidelines](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines)
- [Mark Seemann -- Dependency Injection Principles, Practices, and Patterns (book)](https://www.manning.com/books/dependency-injection-principles-practices-and-patterns)
