---
tags:
  - csharp
  - asp-net-core
  - configuration
  - options-pattern
---


This is the most important decision when using the Options pattern: which interface to inject. Each has different **lifetime** and **reload behavior**.

### IOptions\<T\>

```csharp
public class MyService
{
    private readonly MySettings _settings;

    public MyService(IOptions<MySettings> options)
    {
        // .Value is read once and cached for the app's lifetime
        _settings = options.Value;
    }
}
```

- Registered as a **Singleton**
- Reads configuration **once** at first access, then caches it forever
- Does **not** support config reload -- changes to `appsettings.json` at runtime are ignored
- The simplest and most common choice

### IOptionsSnapshot\<T\>

```csharp
public class MyController : ControllerBase
{
    private readonly MySettings _settings;

    public MyController(IOptionsSnapshot<MySettings> options)
    {
        // .Value reflects the config as of the start of this request
        _settings = options.Value;
    }
}
```

- Registered as **Scoped** (one instance per HTTP request / DI scope)
- Re-reads configuration **on each new scope** (typically each HTTP request)
- Supports config reload -- if `appsettings.json` changes, the next request gets the new values
- Cannot be injected into **Singleton** services (would cause a captive dependency)

### IOptionsMonitor\<T\>

```csharp
public class BackgroundWorker : BackgroundService
{
    private readonly IOptionsMonitor<MySettings> _monitor;

    public BackgroundWorker(IOptionsMonitor<MySettings> monitor)
    {
        _monitor = monitor;

        // Subscribe to changes
        _monitor.OnChange(settings =>
        {
            Console.WriteLine($"Config changed! New host: {settings.Host}");
        });
    }

    protected override async Task ExecuteAsync(CancellationToken ct)
    {
        while (!ct.IsCancellationRequested)
        {
            // .CurrentValue always returns the latest configuration
            var settings = _monitor.CurrentValue;
            await DoWork(settings, ct);
            await Task.Delay(TimeSpan.FromMinutes(1), ct);
        }
    }
}
```

- Registered as a **Singleton**
- `.CurrentValue` always returns the **latest** configuration (not cached like `IOptions<T>`)
- Provides `OnChange(Action<T>)` callback for reactive updates
- Safe to inject into **Singleton** services
- Best for long-running services (background workers, hosted services) that need to react to config changes

### Comparison Table

| Feature | `IOptions<T>` | `IOptionsSnapshot<T>` | `IOptionsMonitor<T>` |
|---|---|---|---|
| **DI Lifetime** | Singleton | Scoped | Singleton |
| **Reads config** | Once (first access) | Per scope / request | On every `.CurrentValue` access |
| **Supports reload** | No | Yes | Yes |
| **Change notification** | No | No | Yes (`OnChange`) |
| **Named options** | No | Yes (`.Get(name)`) | Yes (`.Get(name)`) |
| **Inject into Singleton** | Yes | **No** (captive dependency) | Yes |
| **Best for** | Static config that never changes | Per-request config in controllers/services | Background services, long-running singletons |
| **Access property** | `.Value` | `.Value` | `.CurrentValue` |

> [!danger] Captive Dependency Warning
> Never inject `IOptionsSnapshot<T>` into a Singleton service. The Scoped instance gets "captured" by the Singleton and never refreshes, silently defeating the purpose of snapshot semantics. ASP.NET Core will throw an `InvalidOperationException` if `ValidateScopes` is enabled (which it is by default in Development).

> [!tip] Pro Tip
> **When in doubt, use `IOptions<T>`**. Most configuration does not change at runtime, and `IOptions<T>` is the simplest. Only reach for `IOptionsSnapshot<T>` or `IOptionsMonitor<T>` when you have a concrete requirement for runtime reload.

> [!summary] Section Summary
> `IOptions<T>` is a singleton that reads once. `IOptionsSnapshot<T>` is scoped and re-reads per request. `IOptionsMonitor<T>` is a singleton with live updates and change notifications. Choose based on whether your config changes at runtime and whether the consumer is scoped or singleton.
