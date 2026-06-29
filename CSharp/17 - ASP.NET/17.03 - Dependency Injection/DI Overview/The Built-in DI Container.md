---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - fundamentals
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
