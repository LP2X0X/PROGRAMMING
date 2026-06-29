---
tags:
  - csharp
  - asp-net-core
  - configuration
  - binding
  - strongly-typed
---


> [!abstract] Overview
> ASP.NET Core allows you to **bind raw configuration data** (from JSON files, environment variables, command-line args, etc.) directly into **strongly typed C# classes**. This eliminates magic strings, enables IntelliSense, and makes configuration testable. Combined with **data annotation validation** and **change-token watching**, strongly typed configuration is the production-grade approach for managing application settings.

**Configuration binding** is the process of mapping a configuration section (e.g., a JSON object) to a plain C# class. The property names in the class must match the key names in the configuration source.

### The Settings Class

```csharp
public class SmtpSettings
{
    public string Host { get; set; } = string.Empty;
    public int Port { get; set; }
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
    public bool UseSsl { get; set; }
}
```

### The Configuration Source (appsettings.json)

```json
{
  "Smtp": {
    "Host": "smtp.example.com",
    "Port": 587,
    "Username": "noreply@example.com",
    "Password": "s3cret",
    "UseSsl": true
  }
}
```

### Binding with GetSection().Get\<T\>()

```csharp
var builder = WebApplication.CreateBuilder(args);

// One-shot binding: reads config once, returns a populated object
SmtpSettings smtp = builder.Configuration
    .GetSection("Smtp")
    .Get<SmtpSettings>()!;

Console.WriteLine(smtp.Host); // "smtp.example.com"
Console.WriteLine(smtp.Port); // 587
```

> [!ad-note]
> `Get<T>()` returns `null` if the section does not exist. Use the null-forgiving operator (`!`) only when you are certain the section is present, or handle the null case explicitly.

### Binding with Bind()

```csharp
var smtpSettings = new SmtpSettings();
builder.Configuration.GetSection("Smtp").Bind(smtpSettings);
```

The `Bind()` method populates an **existing instance** rather than creating a new one. This is useful when you need to set defaults on the object before binding overlays the configuration values.

> [!warning] Common Misconception
> `Get<T>()` and `Bind()` produce a **snapshot** of the configuration at the time of the call. They do **not** automatically update when the underlying configuration source changes. For live-reloading behavior, use the [[Options Pattern]] with `IOptionsMonitor<T>`.

> [!summary] Section Summary
> Use `GetSection("Key").Get<T>()` for one-shot binding that returns a new object, or `Bind()` to populate an existing instance. Both produce static snapshots and do not track configuration changes.
