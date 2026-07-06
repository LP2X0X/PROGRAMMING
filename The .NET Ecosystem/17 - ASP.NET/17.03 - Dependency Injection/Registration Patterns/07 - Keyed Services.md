---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
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
