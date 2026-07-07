---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - pitfalls
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
