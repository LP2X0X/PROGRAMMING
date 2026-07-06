---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
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
> You might think you need a factory delegate every time a constructor takes a `string` or `int` parameter. Often, a better approach is to use the [[Registering Options|Options pattern]] and inject `IOptions<T>` into the constructor, keeping the registration clean. Reserve factory delegates for cases where the construction logic is genuinely conditional or complex.

> [!caution]
> Avoid calling `sp.GetRequiredService<T>()` to resolve services with a **shorter lifetime** than the one you are registering. For example, resolving a scoped service inside a singleton factory will capture a single scoped instance forever, leading to the **captive dependency** problem. See [[Common DI Pitfalls]] for details.

> [!summary] Section Summary
> - Factory registration passes a `Func<IServiceProvider, TService>` delegate to the `Add{Lifetime}` method.
> - Use `sp.GetRequiredService<T>()` inside the delegate to resolve other dependencies.
> - Appropriate when you need conditional logic, non-service constructor parameters, or complex initialization.
> - Avoid capturing shorter-lived services inside longer-lived registrations (captive dependency).
> - Consider the Options pattern as an alternative for simple configuration values.
