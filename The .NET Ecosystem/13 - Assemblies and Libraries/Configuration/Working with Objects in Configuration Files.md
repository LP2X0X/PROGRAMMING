---
tags:
  - csharp
  - configuration
---

## The Options Pattern

Instead of reading individual configuration values with string keys, the **Options pattern** maps entire JSON sections to strongly-typed C# classes. This gives you compile-time safety, IntelliSense, and cleaner code.

---

## Step 1: Define Configuration in appsettings.json

```json
{
  "SmtpSettings": {
    "Server": "smtp.example.com",
    "Port": 587,
    "SenderEmail": "noreply@example.com"
  }
}
```

---

## Step 2: Create a Matching C# Class

Property names must match the JSON keys (case-insensitive by default):

```csharp
public class SmtpSettings
{
    public string Server { get; set; } = string.Empty;
    public int Port { get; set; }
    public string SenderEmail { get; set; } = string.Empty;
}
```

```ad-note
title: Naming Convention
The class name does not need to match the JSON section name — the mapping happens when you call `GetSection("SmtpSettings")`. However, keeping them consistent is a strong convention.
```

---

## Step 3: Register in Program.cs (Modern .NET 6+)

```csharp
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection("SmtpSettings"));
```

This registers `IOptions<SmtpSettings>`, `IOptionsSnapshot<SmtpSettings>`, and `IOptionsMonitor<SmtpSettings>` in the DI container automatically.

---

## Step 4: Inject with IOptions<T>

```csharp
public class EmailService
{
    private readonly SmtpSettings _settings;

    public EmailService(IOptions<SmtpSettings> options)
    {
        _settings = options.Value; // unwrap the options wrapper
    }

    public void Send(string to, string subject)
    {
        // use _settings.Server, _settings.Port, etc.
    }
}
```

```ad-warning
title: Don't Inject the Class Directly
Inject `IOptions<SmtpSettings>`, not `SmtpSettings`. The DI container does not register the raw class — only the `IOptions<T>` wrappers.
```

---

## IOptions vs IOptionsSnapshot vs IOptionsMonitor

| Interface | DI Lifetime | Reloads on change? | Use when |
|---|---|---|---|
| `IOptions<T>` | Singleton | No | Values never change at runtime |
| `IOptionsSnapshot<T>` | Scoped | Yes (per request) | Web apps needing fresh config each request |
| `IOptionsMonitor<T>` | Singleton | Yes (live callback) | Background services reacting to config changes |

```ad-info
title: When to Use IOptionsMonitor
`IOptionsMonitor<T>` exposes an `OnChange` callback, making it ideal for long-running hosted services that need to react to `appsettings.json` edits without restarting.

builder.Configuration sources must have `reloadOnChange: true` (the default for JSON files).
```

---

## See Also

- [[Bind and Get Methods]]
- [[Configuring Applications with Configuration Files]]
- [[Multiple Configuration Files]]
