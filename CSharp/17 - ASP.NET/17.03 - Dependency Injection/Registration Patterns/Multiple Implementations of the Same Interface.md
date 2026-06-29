---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
---

## Multiple Implementations of the Same Interface

You can register **multiple concrete types** against the same interface. To consume all of them, inject `IEnumerable<TService>`.

### Registration

```csharp
builder.Services.AddScoped<INotifier, EmailNotifier>();
builder.Services.AddScoped<INotifier, SmsNotifier>();
builder.Services.AddScoped<INotifier, PushNotifier>();
builder.Services.AddScoped<INotifier, SlackNotifier>();
```

### The interface and implementations

```csharp
public interface INotifier
{
    Task SendAsync(string userId, string message);
}

public class EmailNotifier : INotifier
{
    public async Task SendAsync(string userId, string message)
    {
        // Send email notification
    }
}

public class SmsNotifier : INotifier
{
    public async Task SendAsync(string userId, string message)
    {
        // Send SMS notification
    }
}

public class PushNotifier : INotifier
{
    public async Task SendAsync(string userId, string message)
    {
        // Send push notification
    }
}

public class SlackNotifier : INotifier
{
    public async Task SendAsync(string userId, string message)
    {
        // Send Slack message
    }
}
```

### Consuming all implementations

```csharp
public class NotificationDispatcher
{
    private readonly IEnumerable<INotifier> _notifiers;

    public NotificationDispatcher(IEnumerable<INotifier> notifiers)
    {
        _notifiers = notifiers;
    }

    public async Task NotifyAllChannelsAsync(string userId, string message)
    {
        // Iterates over EmailNotifier, SmsNotifier, PushNotifier, SlackNotifier
        foreach (var notifier in _notifiers)
        {
            await notifier.SendAsync(userId, message);
        }
    }
}
```

> [!ad-note]
> If you inject `INotifier` (singular, not `IEnumerable<INotifier>`), you get the **last registered** implementation -- in this case, `SlackNotifier`. This is because each call to `AddScoped<INotifier, T>()` adds a new registration, and the container resolves the last one for singular injection. This behavior is intentional and is how [[TryAdd Methods]] work.

> [!summary] Section Summary
> - Multiple implementations of the same interface are registered with repeated `Add{Lifetime}` calls.
> - Inject `IEnumerable<TService>` to receive all registered implementations.
> - Injecting the interface directly (not `IEnumerable`) returns the **last registered** implementation.
> - This pattern is ideal for notification systems, validation pipelines, and plugin architectures.
> - With .NET 8+, [[Keyed Services|keyed services]] offer a more targeted alternative when you need a specific implementation rather than all of them.
