---
tags:
  - csharp
  - asp-net-core
  - configuration
  - binding
  - strongly-typed
---


JSON nesting maps directly to **nested C# classes**. The binder walks the property tree recursively.

### Configuration (appsettings.json)

```json
{
  "Email": {
    "Smtp": {
      "Host": "smtp.example.com",
      "Port": 587
    },
    "Templates": {
      "WelcomeSubject": "Welcome aboard!",
      "ResetPasswordSubject": "Reset your password"
    }
  }
}
```

### C# Classes

```csharp
public class EmailSettings
{
    public SmtpConfig Smtp { get; set; } = new();
    public TemplateConfig Templates { get; set; } = new();
}

public class SmtpConfig
{
    public string Host { get; set; } = string.Empty;
    public int Port { get; set; }
}

public class TemplateConfig
{
    public string WelcomeSubject { get; set; } = string.Empty;
    public string ResetPasswordSubject { get; set; } = string.Empty;
}
```

### Registration

```csharp
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("Email"));
```

### Consumption

```csharp
public class WelcomeMailer
{
    private readonly EmailSettings _email;

    public WelcomeMailer(IOptions<EmailSettings> options)
    {
        _email = options.Value;
    }

    public void Send()
    {
        var host = _email.Smtp.Host;           // "smtp.example.com"
        var subject = _email.Templates.WelcomeSubject; // "Welcome aboard!"
    }
}
```

> [!info] Definition
> **Nested binding** works because the configuration binder uses reflection to walk the object graph. Each property that is itself a complex type triggers a recursive bind into the corresponding configuration sub-section.

> [!warning] Common Misconception
> Nested objects must be **initialized** (e.g., `= new()`) or the binder will encounter a null reference. If the nested property is null, binding silently skips it and the nested values are lost.

> [!summary] Section Summary
> JSON nesting maps to nested C# classes. Initialize nested properties to avoid null references. The binder recursively walks the object graph.
