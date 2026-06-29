---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
---

## Registering Options

The **Options pattern** is the standard way to bind configuration sections (from `appsettings.json`, environment variables, etc.) to strongly-typed classes and inject them into services.

### 1. Define the settings class

```csharp
public class SmtpSettings
{
    public string Host { get; set; } = string.Empty;
    public int Port { get; set; } = 587;
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
    public bool EnableSsl { get; set; } = true;
    public string FromAddress { get; set; } = string.Empty;
}
```

### 2. Add the configuration section to appsettings.json

```json
{
  "Smtp": {
    "Host": "smtp.company.com",
    "Port": 587,
    "Username": "notifications@company.com",
    "Password": "secret-from-key-vault",
    "EnableSsl": true,
    "FromAddress": "no-reply@company.com"
  }
}
```

### 3. Register the options binding

```csharp
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection("Smtp"));
```

### 4. Inject and use the options

```csharp
public class EmailNotificationService
{
    private readonly SmtpSettings _settings;

    public EmailNotificationService(IOptions<SmtpSettings> options)
    {
        _settings = options.Value;
    }

    public async Task SendAsync(string to, string subject, string body)
    {
        using var client = new SmtpClient(_settings.Host, _settings.Port);
        client.EnableSsl = _settings.EnableSsl;
        client.Credentials = new NetworkCredential(
            _settings.Username, _settings.Password);

        var message = new MailMessage(_settings.FromAddress, to, subject, body);
        await client.SendMailAsync(message);
    }
}
```

### IOptions vs IOptionsSnapshot vs IOptionsMonitor

| Interface | Lifetime | Reloads on Change | Use When |
|---|---|---|---|
| `IOptions<T>` | Singleton | No | Config read once at startup |
| `IOptionsSnapshot<T>` | Scoped | Yes, per request | Config may change; scoped services |
| `IOptionsMonitor<T>` | Singleton | Yes, via `OnChange` callback | Singleton services that need live updates |

```csharp
// For a scoped service that should pick up config changes on each request
public class ReportGenerator
{
    private readonly ReportSettings _settings;

    public ReportGenerator(IOptionsSnapshot<ReportSettings> options)
    {
        _settings = options.Value; // Fresh value each request
    }
}

// For a singleton service that needs to react to config changes
public class BackgroundEmailSender
{
    private SmtpSettings _settings;

    public BackgroundEmailSender(IOptionsMonitor<SmtpSettings> monitor)
    {
        _settings = monitor.CurrentValue;

        monitor.OnChange(newSettings =>
        {
            _settings = newSettings;
        });
    }
}
```

### Options validation

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()  // Validates [Required], [Range], etc.
    .ValidateOnStart();         // Fail fast at startup, not on first use
```

```csharp
public class SmtpSettings
{
    [Required]
    public string Host { get; set; } = string.Empty;

    [Range(1, 65535)]
    public int Port { get; set; } = 587;

    [Required, EmailAddress]
    public string FromAddress { get; set; } = string.Empty;
}
```

> [!tip]
> Always call `.ValidateOnStart()` so that misconfigured settings cause an immediate startup failure rather than a runtime error when the service is first used. This is especially important in production -- you want to fail during deployment, not during a customer request.

> [!summary] Section Summary
> - Use `services.Configure<T>(config.GetSection("..."))` to bind configuration to a strongly-typed class.
> - Inject `IOptions<T>` for static config, `IOptionsSnapshot<T>` for per-request refresh, `IOptionsMonitor<T>` for live singleton updates.
> - Use `AddOptions<T>().BindConfiguration().ValidateDataAnnotations().ValidateOnStart()` for validated, fail-fast configuration.
> - The Options pattern replaces the need for factory delegates that read configuration manually.
> - This is the standard ASP.NET Core approach -- prefer it over injecting `IConfiguration` directly into services.
