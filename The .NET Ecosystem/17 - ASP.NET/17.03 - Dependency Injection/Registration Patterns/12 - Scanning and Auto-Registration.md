---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
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
