---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
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
