---
tags:
  - csharp
  - asp-net-core
  - configuration
  - options-pattern
---


**Named Options** solve the problem of needing multiple configurations of the same type. For example, your application might connect to two different SMTP servers -- one for transactional emails and one for marketing emails.

### The Problem

With basic `Configure<T>`, you can only have one `SmtpSettings`. What if you need two?

### Configuration

```json
{
  "SmtpServers": {
    "Transactional": {
      "Host": "smtp-transactional.example.com",
      "Port": 587,
      "Username": "txn@example.com"
    },
    "Marketing": {
      "Host": "smtp-marketing.example.com",
      "Port": 465,
      "Username": "marketing@example.com"
    }
  }
}
```

### Registration with Names

```csharp
builder.Services.Configure<SmtpSettings>("Transactional",
    builder.Configuration.GetSection("SmtpServers:Transactional"));

builder.Services.Configure<SmtpSettings>("Marketing",
    builder.Configuration.GetSection("SmtpServers:Marketing"));
```

### Consuming Named Options

Named options are accessed through `IOptionsSnapshot<T>` or `IOptionsMonitor<T>` using the `.Get(name)` method:

```csharp
public class EmailService
{
    private readonly SmtpSettings _transactional;
    private readonly SmtpSettings _marketing;

    public EmailService(IOptionsSnapshot<SmtpSettings> options)
    {
        _transactional = options.Get("Transactional");
        _marketing = options.Get("Marketing");
    }

    public async Task SendTransactionalAsync(string to, string subject, string body)
    {
        await SendViaServer(_transactional, to, subject, body);
    }

    public async Task SendMarketingAsync(string to, string subject, string body)
    {
        await SendViaServer(_marketing, to, subject, body);
    }

    private async Task SendViaServer(SmtpSettings settings, string to, string subject, string body)
    {
        using var client = new SmtpClient(settings.Host, settings.Port);
        // ... send logic
    }
}
```

> [!warning] Common Misconception
> `IOptions<T>` does **not** support named options. Calling `IOptions<T>.Value` always returns the **default** (unnamed) configuration. You must use `IOptionsSnapshot<T>.Get(name)` or `IOptionsMonitor<T>.Get(name)` to access named instances.

### Using Constants for Names

Avoid magic strings by defining name constants:

```csharp
public static class SmtpOptionNames
{
    public const string Transactional = nameof(Transactional);
    public const string Marketing = nameof(Marketing);
}

// Registration
builder.Services.Configure<SmtpSettings>(
    SmtpOptionNames.Transactional,
    builder.Configuration.GetSection("SmtpServers:Transactional"));

// Consumption
var txnSettings = options.Get(SmtpOptionNames.Transactional);
```

> [!summary] Section Summary
> Named options allow multiple configurations of the same type, identified by a string name. Register with `Configure<T>(name, section)` and resolve with `IOptionsSnapshot<T>.Get(name)` or `IOptionsMonitor<T>.Get(name)`. `IOptions<T>` does not support named options.
