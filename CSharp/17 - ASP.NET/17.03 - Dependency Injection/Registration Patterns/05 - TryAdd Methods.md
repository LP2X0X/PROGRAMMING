---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
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
