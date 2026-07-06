---
tags:
  - csharp
  - asp-net-core
  - configuration
  - binding
  - strongly-typed
---


**Change tokens** allow you to react to configuration file changes at runtime. When `reloadOnChange: true` is set (the default for JSON files), the configuration system watches the file and triggers reload tokens.

### IConfiguration.GetReloadToken()

```csharp
var token = builder.Configuration.GetReloadToken();
token.RegisterChangeCallback(state =>
{
    Console.WriteLine("Configuration has been reloaded!");
    // Re-read values, update caches, etc.
}, state: null);
```

> [!warning] Common Misconception
> `GetReloadToken()` returns a **one-time-use** token. After it fires, you must call `GetReloadToken()` again to get a new token for subsequent changes. For continuous watching, use `ChangeToken.OnChange()`.

### ChangeToken.OnChange() for Continuous Watching

```csharp
using Microsoft.Extensions.Primitives;

ChangeToken.OnChange(
    () => builder.Configuration.GetReloadToken(),
    () =>
    {
        Console.WriteLine("Configuration changed! Refreshing...");
        // This callback fires every time the config file is modified
    });
```

### How This Relates to IOptionsMonitor\<T\>

`IOptionsMonitor<T>` uses change tokens internally. When you inject `IOptionsMonitor<T>`, it automatically subscribes to configuration changes and provides updated values:

```csharp
public class DynamicEmailService
{
    private readonly IOptionsMonitor<SmtpSettings> _optionsMonitor;

    public DynamicEmailService(IOptionsMonitor<SmtpSettings> optionsMonitor)
    {
        _optionsMonitor = optionsMonitor;

        // React to changes
        _optionsMonitor.OnChange(newSettings =>
        {
            Console.WriteLine($"SMTP host changed to: {newSettings.Host}");
        });
    }

    public void Send()
    {
        // Always gets the latest values
        var current = _optionsMonitor.CurrentValue;
        Console.WriteLine($"Sending via {current.Host}:{current.Port}");
    }
}
```

### The Three Options Interfaces

| Interface | Lifetime | Reloads | Use Case |
|---|---|---|---|
| `IOptions<T>` | Singleton | No | Settings that never change at runtime |
| `IOptionsSnapshot<T>` | Scoped | Per-request | Web apps: fresh config per HTTP request |
| `IOptionsMonitor<T>` | Singleton | Yes (live) | Background services, long-lived consumers |

> [!danger] Critical Warning
> Do **not** inject `IOptionsSnapshot<T>` into a singleton service. It is scoped and will throw a runtime error or silently give stale data depending on your DI container validation settings.

> [!summary] Section Summary
> `GetReloadToken()` provides one-shot change notification. `ChangeToken.OnChange()` provides continuous watching. In practice, prefer `IOptionsMonitor<T>` which handles change tokens internally and always serves the latest configuration.
