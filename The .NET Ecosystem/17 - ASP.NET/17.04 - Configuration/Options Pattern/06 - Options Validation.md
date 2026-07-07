---
tags:
  - csharp
  - asp-net-core
  - configuration
  - options-pattern
---


Configuration errors are among the most frustrating bugs because they often manifest as cryptic runtime failures deep in the application. **Options validation** catches them at startup.

### Data Annotations Validation

The simplest approach -- decorate your POCO with `System.ComponentModel.DataAnnotations` attributes:

```csharp
using System.ComponentModel.DataAnnotations;

public class SmtpSettings
{
    public const string SectionName = "Smtp";

    [Required(ErrorMessage = "SMTP host is required")]
    public string Host { get; set; } = string.Empty;

    [Range(1, 65535, ErrorMessage = "Port must be between 1 and 65535")]
    public int Port { get; set; } = 587;

    [Required]
    [EmailAddress(ErrorMessage = "Username must be a valid email address")]
    public string Username { get; set; } = string.Empty;

    [Required]
    [MinLength(8, ErrorMessage = "Password must be at least 8 characters")]
    public string Password { get; set; } = string.Empty;

    public bool EnableSsl { get; set; } = true;
}
```

### Registering Validation

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .Bind(builder.Configuration.GetSection(SmtpSettings.SectionName))
    .ValidateDataAnnotations()  // Validates using [Required], [Range], etc.
    .ValidateOnStart();         // Validate immediately at app startup
```

> [!danger] Critical: ValidateOnStart
> Without `.ValidateOnStart()`, validation only runs **the first time** the options are resolved from DI. If nothing requests `SmtpSettings` until a user sends an email 3 hours after deployment, that is when your app discovers the config is broken. **Always add `.ValidateOnStart()`** to fail fast.

### Custom Validation with Validate()

For validation logic that goes beyond data annotations:

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .Bind(builder.Configuration.GetSection(SmtpSettings.SectionName))
    .ValidateDataAnnotations()
    .Validate(settings =>
    {
        // Custom rule: if SSL is enabled, port must be 465 or 587
        if (settings.EnableSsl && settings.Port != 465 && settings.Port != 587)
            return false;
        return true;
    }, "When SSL is enabled, port must be 465 or 587")
    .ValidateOnStart();
```

### Validation with IValidateOptions\<T\>

For complex validation that needs dependency injection (e.g., checking a database):

```csharp
public class SmtpSettingsValidator : IValidateOptions<SmtpSettings>
{
    public ValidateOptionsResult Validate(string? name, SmtpSettings options)
    {
        var failures = new List<string>();

        if (string.IsNullOrWhiteSpace(options.Host))
            failures.Add("SMTP Host is required.");

        if (options.Port is < 1 or > 65535)
            failures.Add($"Port {options.Port} is outside the valid range (1-65535).");

        if (options.EnableSsl && options.Port == 25)
            failures.Add("Port 25 does not support SSL. Use 465 or 587.");

        if (options.Timeout < TimeSpan.FromSeconds(5))
            failures.Add("Timeout must be at least 5 seconds.");

        return failures.Count > 0
            ? ValidateOptionsResult.Fail(failures)
            : ValidateOptionsResult.Success;
    }
}

// Registration
builder.Services.AddSingleton<IValidateOptions<SmtpSettings>, SmtpSettingsValidator>();
```

### Validation Approaches Comparison

| Approach | Complexity | DI Support | Best For |
|---|---|---|---|
| `ValidateDataAnnotations()` | Low | No | Simple required/range/regex checks |
| `.Validate(Func<T, bool>)` | Medium | No | Cross-property rules within the same class |
| `IValidateOptions<T>` | High | Yes | Complex rules, external dependencies, reusable validators |

> [!tip] Pro Tip
> In .NET 8+, you can use the `[OptionsValidator]` source generator attribute on a class implementing `IValidateOptions<T>` to get compile-time validation code generation, eliminating reflection overhead from data annotations.

> [!summary] Section Summary
> Always validate options to catch configuration errors at startup. Use `ValidateDataAnnotations()` for simple rules, `.Validate()` lambdas for cross-property checks, and `IValidateOptions<T>` for complex/injectable validation. **Always add `.ValidateOnStart()`**.
