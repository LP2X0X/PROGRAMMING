---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
---

## Instance Registration

You can hand a **pre-built object** directly to the container. This is only available for singletons (since the container must return the same instance every time).

```csharp
var smtpSettings = new SmtpSettings
{
    Host = "smtp.company.com",
    Port = 587,
    EnableSsl = true
};

builder.Services.AddSingleton(smtpSettings);

// You can also register it behind an interface
var notifier = new EmailNotificationService(smtpSettings);
builder.Services.AddSingleton<INotificationService>(notifier);
```

> [!danger]
> When you register a pre-built instance, the container does **not** manage its lifecycle. If the object implements `IDisposable`, the container will **not** call `Dispose()` on it when the application shuts down. You are responsible for disposing it yourself (or registering it with a factory delegate instead, which does get disposed).

```csharp
// Container WILL dispose this (factory delegate registration)
builder.Services.AddSingleton<INotificationService>(sp =>
    new EmailNotificationService(sp.GetRequiredService<SmtpSettings>()));

// Container will NOT dispose this (instance registration)
var service = new EmailNotificationService(smtpSettings);
builder.Services.AddSingleton<INotificationService>(service);
```

### When to use instance registration

- **Configuration objects** that you build up manually before the container is constructed.
- **Pre-initialized singletons** that must exist before the DI container is built (rare).
- **Test doubles** in integration tests where you want to inject a specific mock.

> [!summary] Section Summary
> - Instance registration uses `AddSingleton(instance)` or `AddSingleton<TService>(instance)`.
> - Only available for singletons since the same object is returned every time.
> - The container does **not** dispose instance-registered objects -- you must handle disposal yourself.
> - Prefer factory delegate registration if the object implements `IDisposable`.
> - Common for configuration objects and test doubles.
