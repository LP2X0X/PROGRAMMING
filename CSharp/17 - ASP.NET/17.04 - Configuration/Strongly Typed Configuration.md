---
tags: [csharp, asp-net-core, configuration, binding, strongly-typed]
date: 2026-06-18
aliases: [Strongly-Typed Config, Options Binding, Configuration Binding]
status: complete
---

# Strongly Typed Configuration

> [!abstract] Overview
> ASP.NET Core allows you to **bind raw configuration data** (from JSON files, environment variables, command-line args, etc.) directly into **strongly typed C# classes**. This eliminates magic strings, enables IntelliSense, and makes configuration testable. Combined with **data annotation validation** and **change-token watching**, strongly typed configuration is the production-grade approach for managing application settings.

---

## Table of Contents

- [[#Binding Configuration to Classes]]
- [[#Configure vs Get -- When to Use Which]]
- [[#Nested Configuration Objects]]
- [[#Array and List Binding]]
- [[#Complex Binding Example]]
- [[#Data Annotation Validation on Options]]
- [[#ValidateDataAnnotations and ValidateOnStart]]
- [[#Custom Validation Logic]]
- [[#Configuration Change Tokens]]
- [[#Binding to Records]]
- [[#Real-World Full Application Configuration]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]
- [[#Further Reading]]

---

## Binding Configuration to Classes

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

---

## Configure vs Get -- When to Use Which

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

---

## Nested Configuration Objects

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

---

## Array and List Binding

JSON arrays bind to `List<T>`, `T[]`, `IEnumerable<T>`, `IList<T>`, or `IReadOnlyList<T>`.

### Configuration (appsettings.json)

```json
{
  "AllowedOrigins": [
    "https://app.example.com",
    "https://admin.example.com"
  ],
  "DatabaseConnections": [
    {
      "Name": "Primary",
      "ConnectionString": "Server=db1;Database=App;..."
    },
    {
      "Name": "ReadReplica",
      "ConnectionString": "Server=db2;Database=App;..."
    }
  ]
}
```

### C# Classes

```csharp
public class CorsSettings
{
    public List<string> AllowedOrigins { get; set; } = new();
}

public class DatabaseSettings
{
    public List<DatabaseConnection> DatabaseConnections { get; set; } = new();
}

public class DatabaseConnection
{
    public string Name { get; set; } = string.Empty;
    public string ConnectionString { get; set; } = string.Empty;
}
```

### Registration and Usage

```csharp
builder.Services.Configure<CorsSettings>(builder.Configuration);
builder.Services.Configure<DatabaseSettings>(builder.Configuration);
```

```csharp
public class StartupValidator
{
    public StartupValidator(IOptions<DatabaseSettings> options)
    {
        foreach (var db in options.Value.DatabaseConnections)
        {
            Console.WriteLine($"{db.Name}: {db.ConnectionString}");
        }
    }
}
```

> [!tip] Pro Tip
> Arrays in JSON are indexed by position (0, 1, 2, ...). You can also override individual array elements using the key format `Section:0:Property` in environment variables:
> ```bash
> export DatabaseConnections__0__Name="Overridden"
> ```

### Dictionary Binding

Dictionaries also work. A JSON object with arbitrary keys maps to `Dictionary<string, T>`:

```json
{
  "FeatureFlags": {
    "DarkMode": true,
    "BetaSearch": false,
    "NewDashboard": true
  }
}
```

```csharp
public class FeatureFlagSettings
{
    public Dictionary<string, bool> FeatureFlags { get; set; } = new();
}
```

> [!summary] Section Summary
> JSON arrays bind to `List<T>`, `T[]`, or any `IEnumerable<T>` implementation. Dictionaries bind to `Dictionary<string, T>`. Initialize collections to avoid null references.

---

## Complex Binding Example

Here is a realistic configuration demonstrating nested objects, lists, enums, and dictionaries all in one structure.

### Configuration (appsettings.json)

```json
{
  "AppSettings": {
    "ApplicationName": "OrderProcessor",
    "Version": "2.4.1",
    "Environment": "Production",
    "Retry": {
      "MaxAttempts": 3,
      "DelayMilliseconds": 1000,
      "BackoffStrategy": "Exponential"
    },
    "Endpoints": [
      {
        "Name": "OrderApi",
        "BaseUrl": "https://api.orders.example.com",
        "TimeoutSeconds": 30,
        "Headers": {
          "X-Api-Key": "abc123",
          "X-Client-Id": "order-processor"
        }
      },
      {
        "Name": "InventoryApi",
        "BaseUrl": "https://api.inventory.example.com",
        "TimeoutSeconds": 15,
        "Headers": {
          "X-Api-Key": "def456"
        }
      }
    ],
    "NotificationChannels": [ "Email", "Slack", "Sms" ]
  }
}
```

### C# Classes

```csharp
public enum BackoffStrategy
{
    Linear,
    Exponential,
    Constant
}

public class AppSettings
{
    public string ApplicationName { get; set; } = string.Empty;
    public string Version { get; set; } = string.Empty;
    public string Environment { get; set; } = string.Empty;
    public RetrySettings Retry { get; set; } = new();
    public List<EndpointSettings> Endpoints { get; set; } = new();
    public List<string> NotificationChannels { get; set; } = new();
}

public class RetrySettings
{
    public int MaxAttempts { get; set; }
    public int DelayMilliseconds { get; set; }
    public BackoffStrategy BackoffStrategy { get; set; }
}

public class EndpointSettings
{
    public string Name { get; set; } = string.Empty;
    public string BaseUrl { get; set; } = string.Empty;
    public int TimeoutSeconds { get; set; }
    public Dictionary<string, string> Headers { get; set; } = new();
}
```

### Registration

```csharp
builder.Services.Configure<AppSettings>(
    builder.Configuration.GetSection("AppSettings"));
```

> [!info] Definition
> **Enum binding** works by matching the string value in JSON to the enum member name (case-insensitive). `"Exponential"` in JSON binds to `BackoffStrategy.Exponential`. Integer values also work: `1` would bind to `BackoffStrategy.Exponential`.

> [!warning] Common Misconception
> If an enum value in JSON does not match any member name, the binding silently falls back to the **default value** (the first member or `0`). It does not throw. Always validate enum values if correctness is critical.

> [!summary] Section Summary
> Complex configurations with nested objects, lists, enums, and dictionaries all bind cleanly. Enum binding is case-insensitive by name. Always validate enum values since mismatches fail silently.

---

## Data Annotation Validation on Options

You can use **System.ComponentModel.DataAnnotations** attributes on your options classes to enforce configuration correctness at startup.

### Annotated Options Class

```csharp
using System.ComponentModel.DataAnnotations;

public class SmtpSettings
{
    [Required(ErrorMessage = "SMTP Host is required")]
    public string Host { get; set; } = string.Empty;

    [Range(1, 65535, ErrorMessage = "Port must be between 1 and 65535")]
    public int Port { get; set; }

    [Required]
    [EmailAddress(ErrorMessage = "Username must be a valid email")]
    public string Username { get; set; } = string.Empty;

    [Required]
    [MinLength(8, ErrorMessage = "Password must be at least 8 characters")]
    public string Password { get; set; } = string.Empty;

    public bool UseSsl { get; set; }
}
```

### Commonly Used Data Annotations

| Attribute | Purpose | Example |
|---|---|---|
| `[Required]` | Value must be non-null and non-empty | Connection strings, hostnames |
| `[Range(min, max)]` | Numeric value within bounds | Ports, timeouts, retry counts |
| `[Url]` | Must be a valid URL format | API base URLs |
| `[EmailAddress]` | Must be a valid email format | Notification addresses |
| `[MinLength(n)]` | String minimum length | Passwords, API keys |
| `[MaxLength(n)]` | String maximum length | Short codes, identifiers |
| `[RegularExpression]` | Must match a regex pattern | Custom format constraints |
| `[StringLength(max)]` | String length within bounds | General text fields |

> [!example] Annotated URL Configuration
> ```csharp
> public class ApiEndpointSettings
> {
>     [Required]
>     public string Name { get; set; } = string.Empty;
> 
>     [Required]
>     [Url(ErrorMessage = "BaseUrl must be a valid URL")]
>     public string BaseUrl { get; set; } = string.Empty;
> 
>     [Range(1, 300, ErrorMessage = "Timeout must be between 1 and 300 seconds")]
>     public int TimeoutSeconds { get; set; } = 30;
> }
> ```

> [!summary] Section Summary
> Data annotations from `System.ComponentModel.DataAnnotations` can be placed on options class properties. They are only enforced when paired with `ValidateDataAnnotations()` during registration (covered next).

---

## ValidateDataAnnotations and ValidateOnStart

Annotations on their own do **nothing** unless you wire up validation during service registration.

### Basic Validation Registration

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations();
```

> [!danger] Critical Warning
> Without `ValidateDataAnnotations()`, your `[Required]` and `[Range]` attributes are completely ignored at runtime. The application will happily start with invalid configuration. Always pair annotations with explicit validation registration.

### Fail-Fast with ValidateOnStart

By default, options validation only runs **the first time `IOptions<T>.Value` is accessed**. This means a misconfigured app might start successfully and only crash later when the faulty options are first resolved.

**`ValidateOnStart()`** forces validation to run during application startup, so misconfigurations are caught immediately:

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

If validation fails at startup, you get an `OptionsValidationException` with a clear message:

```
Unhandled exception. Microsoft.Extensions.Options.OptionsValidationException:
DataAnnotation validation failed for 'SmtpSettings' members: 'Host' with the
error: 'SMTP Host is required'.
```

> [!tip] Pro Tip
> **Always use `ValidateOnStart()`** in production applications. Failing fast at startup is vastly preferable to discovering bad configuration at 3 AM when a rarely-used code path first tries to read the setting.

### Full Pattern

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")          // Binds to the "Smtp" section
    .ValidateDataAnnotations()           // Enables annotation-based validation
    .ValidateOnStart();                  // Runs validation at startup

builder.Services.AddOptions<DatabaseSettings>()
    .BindConfiguration("Database")
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

> [!ad-note]
> `BindConfiguration("SectionName")` is a shorthand that combines `GetSection()` and `Bind()`. It is equivalent to calling `Configure<T>(config.GetSection("SectionName"))` but works within the `AddOptions<T>()` fluent API.

> [!summary] Section Summary
> `ValidateDataAnnotations()` activates annotation-based validation. `ValidateOnStart()` triggers validation at startup instead of on first access. Always use both together for fail-fast behavior.

---

## Custom Validation Logic

For validation rules that go beyond what data annotations can express, use the **`Validate()` method** in the fluent options API.

### Simple Custom Validation

```csharp
builder.Services.AddOptions<RetrySettings>()
    .BindConfiguration("AppSettings:Retry")
    .ValidateDataAnnotations()
    .Validate(options => options.MaxAttempts > 0,
        "MaxRetries must be positive")
    .Validate(options => options.DelayMilliseconds >= 100,
        "Delay must be at least 100ms")
    .ValidateOnStart();
```

### Multi-Property Validation

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()
    .Validate(options =>
    {
        if (options.UseSsl && options.Port == 25)
            return false; // Port 25 should not be used with SSL
        return true;
    }, "Port 25 cannot be used with SSL enabled. Use port 465 or 587.")
    .ValidateOnStart();
```

### IValidateOptions\<T\> for Complex Scenarios

For validation logic that requires injected services (e.g., checking a database or external system), implement `IValidateOptions<T>`:

```csharp
public class SmtpSettingsValidator : IValidateOptions<SmtpSettings>
{
    public ValidateOptionsResult Validate(string? name, SmtpSettings options)
    {
        var errors = new List<string>();

        if (string.IsNullOrWhiteSpace(options.Host))
            errors.Add("Host is required.");

        if (options.Port is < 1 or > 65535)
            errors.Add("Port must be between 1 and 65535.");

        if (options.UseSsl && options.Port == 25)
            errors.Add("Port 25 is not compatible with SSL.");

        if (!options.Host.Contains('.'))
            errors.Add("Host must be a fully qualified domain name.");

        return errors.Count > 0
            ? ValidateOptionsResult.Fail(errors)
            : ValidateOptionsResult.Success;
    }
}
```

Register the validator:

```csharp
builder.Services.AddSingleton<IValidateOptions<SmtpSettings>,
    SmtpSettingsValidator>();

builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateOnStart();
```

> [!tip] Pro Tip
> `IValidateOptions<T>` validators are resolved from DI, so they can depend on other services. This is the only validation approach that supports dependency injection. The `Validate()` lambda cannot inject services.

### Combining All Validation Approaches

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()                     // Annotations first
    .Validate(o => o.Port != 25 || !o.UseSsl,      // Inline lambda
        "Port 25 + SSL is invalid")
    .ValidateOnStart();                            // Fail fast

// Plus an IValidateOptions<T> registered separately
builder.Services.AddSingleton<
    IValidateOptions<SmtpSettings>,
    SmtpSettingsValidator>();
```

All three validation mechanisms run. If any fails, the aggregated errors are reported.

> [!summary] Section Summary
> Use `Validate()` lambdas for simple cross-property checks. Implement `IValidateOptions<T>` when validation needs injected services. All validation approaches (annotations, lambdas, IValidateOptions) combine and run together.

---

## Configuration Change Tokens

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

---

## Binding to Records

Starting with .NET 6, configuration binding works with **C# records** and **init-only properties**.

### Record-Based Options

```csharp
public record SmtpSettings
{
    public required string Host { get; init; }
    public int Port { get; init; } = 587;
    public required string Username { get; init; }
    public required string Password { get; init; }
    public bool UseSsl { get; init; } = true;
}
```

### Registration (same as with classes)

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

### Why Use Records for Configuration

- **Immutability**: `init` setters prevent accidental modification after binding
- **Value equality**: Two `SmtpSettings` with the same values are considered equal
- **Conciseness**: `required` keyword ensures all mandatory properties are set
- **Pattern matching**: Records work naturally with C# pattern matching

> [!ad-note]
> Records with **positional parameters** (constructor-based syntax like `record SmtpSettings(string Host, int Port)`) do **not** work with configuration binding. The binder requires parameterless construction with settable properties. Use the `{ get; init; }` property syntax instead.

> [!example] Record With Validation
> ```csharp
> public record DatabaseOptions
> {
>     [Required]
>     public required string ConnectionString { get; init; }
> 
>     [Range(1, 300)]
>     public int CommandTimeoutSeconds { get; init; } = 30;
> 
>     [Range(1, 200)]
>     public int MaxPoolSize { get; init; } = 100;
> 
>     public bool EnableDetailedErrors { get; init; } = false;
> }
> ```

> [!summary] Section Summary
> Records with `init` properties work with configuration binding and provide immutability. Avoid positional parameter records (constructor syntax) as they are incompatible with the binder.

---

## Real-World Full Application Configuration

Here is a complete, production-grade example tying together all concepts: nested objects, lists, enums, data annotations, custom validation, and fail-fast startup.

### appsettings.json

```json
{
  "Application": {
    "Name": "OrderManagement",
    "Version": "3.1.0"
  },
  "Database": {
    "ConnectionString": "Server=db.internal;Database=Orders;User=app;Password=secret;",
    "CommandTimeoutSeconds": 30,
    "MaxPoolSize": 100,
    "EnableRetryOnFailure": true
  },
  "Smtp": {
    "Host": "smtp.company.com",
    "Port": 587,
    "Username": "orders@company.com",
    "Password": "smtp-s3cret-pwd",
    "UseSsl": true,
    "DefaultFrom": {
      "Name": "Order System",
      "Address": "orders@company.com"
    }
  },
  "FeatureFlags": {
    "EnableNewCheckout": true,
    "EnableBetaReporting": false,
    "EnableDarkMode": true,
    "MaintenanceMode": false
  },
  "ExternalApis": [
    {
      "Name": "PaymentGateway",
      "BaseUrl": "https://api.payments.example.com/v2",
      "ApiKey": "pay_live_abc123",
      "TimeoutSeconds": 15,
      "RetryPolicy": "Exponential"
    },
    {
      "Name": "ShippingProvider",
      "BaseUrl": "https://api.shipping.example.com/v1",
      "ApiKey": "ship_live_xyz789",
      "TimeoutSeconds": 30,
      "RetryPolicy": "Linear"
    }
  ],
  "RateLimiting": {
    "Enabled": true,
    "RequestsPerMinute": 100,
    "BurstLimit": 20
  }
}
```

### Options Classes

```csharp
using System.ComponentModel.DataAnnotations;

// --- Top-level application metadata ---
public record ApplicationSettings
{
    [Required]
    public required string Name { get; init; }
    [Required]
    public required string Version { get; init; }
}

// --- Database ---
public record DatabaseSettings
{
    [Required(ErrorMessage = "Database connection string is required")]
    public required string ConnectionString { get; init; }

    [Range(1, 300)]
    public int CommandTimeoutSeconds { get; init; } = 30;

    [Range(1, 500)]
    public int MaxPoolSize { get; init; } = 100;

    public bool EnableRetryOnFailure { get; init; } = true;
}

// --- SMTP with nested sender ---
public record SmtpSettings
{
    [Required]
    public required string Host { get; init; }

    [Range(1, 65535)]
    public int Port { get; init; } = 587;

    [Required]
    public required string Username { get; init; }

    [Required, MinLength(8)]
    public required string Password { get; init; }

    public bool UseSsl { get; init; } = true;

    public EmailAddress DefaultFrom { get; init; } = new();
}

public record EmailAddress
{
    [Required]
    public required string Name { get; init; }

    [Required, EmailAddress]
    public required string Address { get; init; }
}

// --- Feature flags as dictionary ---
public record FeatureFlagSettings
{
    public Dictionary<string, bool> Flags { get; init; } = new();

    // Convenience accessors
    public bool IsEnabled(string featureName)
        => Flags.TryGetValue(featureName, out var enabled) && enabled;
}

// --- External API endpoints (list of objects with enum) ---
public enum RetryPolicy { None, Linear, Exponential }

public record ExternalApiSettings
{
    [Required]
    public required string Name { get; init; }

    [Required, Url]
    public required string BaseUrl { get; init; }

    [Required]
    public required string ApiKey { get; init; }

    [Range(1, 120)]
    public int TimeoutSeconds { get; init; } = 30;

    public RetryPolicy RetryPolicy { get; init; } = RetryPolicy.None;
}

// --- Rate limiting ---
public record RateLimitingSettings
{
    public bool Enabled { get; init; }

    [Range(1, 10000)]
    public int RequestsPerMinute { get; init; } = 60;

    [Range(1, 1000)]
    public int BurstLimit { get; init; } = 10;
}

// --- Root config that holds the API list ---
public record ExternalApiListSettings
{
    public List<ExternalApiSettings> ExternalApis { get; init; } = new();
}
```

### Service Registration with Validation

```csharp
var builder = WebApplication.CreateBuilder(args);

// Application metadata
builder.Services.AddOptions<ApplicationSettings>()
    .BindConfiguration("Application")
    .ValidateDataAnnotations()
    .ValidateOnStart();

// Database
builder.Services.AddOptions<DatabaseSettings>()
    .BindConfiguration("Database")
    .ValidateDataAnnotations()
    .Validate(o => !o.ConnectionString.Contains("Password=;"),
        "Connection string must include a non-empty password")
    .ValidateOnStart();

// SMTP
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()
    .Validate(o => !(o.UseSsl && o.Port == 25),
        "Port 25 is not valid with SSL. Use 465 or 587.")
    .ValidateOnStart();

// Feature flags (bound to the whole section as a dictionary)
builder.Services.Configure<FeatureFlagSettings>(options =>
{
    var section = builder.Configuration.GetSection("FeatureFlags");
    options.Flags = section.GetChildren()
        .ToDictionary(x => x.Key, x => bool.Parse(x.Value ?? "false"));
});

// External APIs (list binding)
builder.Services.AddOptions<ExternalApiListSettings>()
    .BindConfiguration("")  // Bind to root since ExternalApis is a root-level array
    .ValidateDataAnnotations()
    .ValidateOnStart();

// Rate limiting
builder.Services.AddOptions<RateLimitingSettings>()
    .BindConfiguration("RateLimiting")
    .ValidateDataAnnotations()
    .Validate(o => o.BurstLimit <= o.RequestsPerMinute,
        "BurstLimit cannot exceed RequestsPerMinute")
    .ValidateOnStart();

var app = builder.Build();
```

### Consuming in a Service

```csharp
public class OrderService
{
    private readonly DatabaseSettings _db;
    private readonly SmtpSettings _smtp;
    private readonly FeatureFlagSettings _features;
    private readonly IOptionsMonitor<RateLimitingSettings> _rateLimiting;

    public OrderService(
        IOptions<DatabaseSettings> dbOptions,
        IOptions<SmtpSettings> smtpOptions,
        IOptions<FeatureFlagSettings> featureOptions,
        IOptionsMonitor<RateLimitingSettings> rateLimitingMonitor)
    {
        _db = dbOptions.Value;
        _smtp = smtpOptions.Value;
        _features = featureOptions.Value;
        _rateLimiting = rateLimitingMonitor;

        // React to rate limiting changes at runtime
        _rateLimiting.OnChange(newSettings =>
        {
            Console.WriteLine(
                $"Rate limit updated: {newSettings.RequestsPerMinute} req/min");
        });
    }

    public void ProcessOrder(int orderId)
    {
        if (_features.IsEnabled("MaintenanceMode"))
        {
            throw new InvalidOperationException(
                "System is in maintenance mode");
        }

        // Use _db.ConnectionString, _smtp.Host, etc.
        Console.WriteLine(
            $"Processing order {orderId} on {_db.ConnectionString}");

        var currentRateLimit = _rateLimiting.CurrentValue;
        Console.WriteLine(
            $"Rate limit: {currentRateLimit.RequestsPerMinute}/min");
    }
}
```

> [!tip] Pro Tip
> In a real application, store sensitive values like connection strings and API keys in **User Secrets** (development) or **environment variables / Azure Key Vault** (production), not in `appsettings.json`. See [[Secrets and Environment Variables]].

> [!summary] Section Summary
> A complete application configuration combines nested objects, lists, enums, dictionaries, data annotations, custom validation, and fail-fast startup. Each concern gets its own options record. Sensitive settings should be stored outside of JSON files.

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Strongly typed configuration** in ASP.NET Core replaces magic strings with compile-time-safe C# classes that are populated automatically from JSON, environment variables, and other providers.
>
> **Key takeaways:**
>
> 1. **Binding** maps configuration sections to C# classes via `Get<T>()`, `Bind()`, or `BindConfiguration()`
> 2. **`services.Configure<T>()`** integrates with DI and unlocks the full [[Options Pattern]] (IOptions, IOptionsSnapshot, IOptionsMonitor)
> 3. **Nested objects** work by recursive property-tree walking; always initialize nested properties
> 4. **Arrays/Lists** bind positionally; dictionaries bind by key
> 5. **Enums** bind by name (case-insensitive) and fail silently on mismatch
> 6. **Data annotations** (`[Required]`, `[Range]`, `[Url]`) enforce constraints when paired with `ValidateDataAnnotations()`
> 7. **`ValidateOnStart()`** is essential for fail-fast behavior; without it, validation only runs on first access
> 8. **Custom validation** uses `Validate()` lambdas or `IValidateOptions<T>` (the latter supports DI)
> 9. **Change tokens** via `GetReloadToken()` enable runtime config watching; `IOptionsMonitor<T>` handles this automatically
> 10. **Records** with `init` properties provide immutable, strongly typed options; avoid positional parameter syntax
>
> **The production pattern:**
> ```csharp
> builder.Services.AddOptions<MySettings>()
>     .BindConfiguration("SectionName")
>     .ValidateDataAnnotations()
>     .ValidateOnStart();
> ```

---

## Related Topics

- [[Configuration Overview]] -- foundational concepts of the ASP.NET Core configuration system
- [[Options Pattern]] -- IOptions\<T\>, IOptionsSnapshot\<T\>, IOptionsMonitor\<T\> in depth
- [[Secrets and Environment Variables]] -- managing sensitive configuration outside of JSON files
- [[Dependency Injection]] -- how Configure\<T\>() integrates with the DI container
- [[Middleware]] -- accessing configuration in middleware components

---

## Further Reading

- Microsoft Docs: Options pattern in ASP.NET Core
- Microsoft Docs: Configuration in ASP.NET Core
- Microsoft Docs: Use multiple environments in ASP.NET Core
- Andrew Lock: Adding validation to strongly typed configuration objects in ASP.NET Core
- Steve Smith: ASP.NET Core Configuration Best Practices
