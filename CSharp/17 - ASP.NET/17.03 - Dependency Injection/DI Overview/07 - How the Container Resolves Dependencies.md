---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - fundamentals
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
