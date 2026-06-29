---
tags:
  - csharp
  - asp-net-core
  - configuration
  - binding
  - strongly-typed
---


There are two primary strategies for consuming configuration in ASP.NET Core: **manual binding** with `Get<T>()` and **DI-integrated binding** with `services.Configure<T>()`.

### services.Configure\<T\>()

```csharp
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection("Smtp"));
```

This registers `IOptions<SmtpSettings>`, `IOptionsSnapshot<SmtpSettings>`, and `IOptionsMonitor<SmtpSettings>` into the DI container. You then inject whichever interface you need:

```csharp
public class EmailService
{
    private readonly SmtpSettings _settings;

    public EmailService(IOptions<SmtpSettings> options)
    {
        _settings = options.Value;
    }

    public void Send(string to, string subject, string body)
    {
        // Use _settings.Host, _settings.Port, etc.
    }
}
```

### config.Get\<T\>()

```csharp
SmtpSettings smtp = builder.Configuration
    .GetSection("Smtp")
    .Get<SmtpSettings>()!;

// Register as a singleton manually
builder.Services.AddSingleton(smtp);
```

### Comparison Table

| Aspect | `services.Configure<T>()` | `config.Get<T>()` |
|---|---|---|
| **DI integration** | Automatic (IOptions/IOptionsSnapshot/IOptionsMonitor) | Manual (you register it yourself) |
| **Live reload** | Yes, via IOptionsMonitor or IOptionsSnapshot | No, produces a snapshot |
| **Validation support** | Yes, with ValidateDataAnnotations() | No built-in validation |
| **Named options** | Yes | No |
| **Use case** | Services that need config throughout the app lifetime | Startup-only config, seeding, or simple scripts |
| **Testability** | Excellent (mock IOptions\<T\>) | Good (pass the object directly) |

> [!tip] Pro Tip
> Prefer `services.Configure<T>()` in almost all production scenarios. It integrates with the [[Options Pattern]], supports validation, and enables live-reload. Use `Get<T>()` only for one-off startup logic where you need the values immediately and they will not change.

> [!summary] Section Summary
> `services.Configure<T>()` registers options into DI with full lifecycle support. `config.Get<T>()` produces a plain object for immediate use. Choose Configure for services, Get for startup-only needs.
