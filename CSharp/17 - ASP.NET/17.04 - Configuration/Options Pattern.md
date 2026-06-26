---
tags: [csharp, asp-net-core, configuration, options-pattern]
date: 2026-06-18
aliases: [Options Pattern, IOptions, IOptionsSnapshot, IOptionsMonitor, Named Options, Options Validation]
status: complete
---

# Options Pattern

> [!abstract] Overview
> The **Options pattern** is a first-class mechanism in ASP.NET Core for binding configuration sections (from `appsettings.json`, environment variables, etc.) to strongly-typed C# classes (POCOs). Instead of scattering magic strings like `Configuration["Smtp:Host"]` throughout your code, you define a class whose properties mirror the configuration keys and let the framework wire them together. This gives you **type safety**, **IntelliSense**, **validation**, and **testability** -- all without coupling your services to `IConfiguration` directly.

---

## Table of Contents

- [Why Not Read Raw Configuration Strings](#why-not-read-raw-configuration-strings)
- [The POCO Class](#the-poco-class)
- [Registering Options with DI](#registering-options-with-di)
- [Injecting Options into Services](#injecting-options-into-services)
- [IOptions vs IOptionsSnapshot vs IOptionsMonitor](#ioptions-vs-ioptionssnapshot-vs-ioptionsmonitor)
- [Named Options](#named-options)
- [Options Validation](#options-validation)
- [PostConfigure](#postconfigure)
- [Real-World Examples](#real-world-examples)
- [Comprehensive Summary](#comprehensive-summary)
- [Related Topics](#related-topics)
- [Further Reading](#further-reading)

---

## Why Not Read Raw Configuration Strings

Before the Options pattern, you might read configuration values like this:

```csharp
public class EmailService
{
    private readonly IConfiguration _config;

    public EmailService(IConfiguration config)
    {
        _config = config;
    }

    public void Send()
    {
        // Magic strings everywhere -- no compile-time safety
        string host = _config["Smtp:Host"];
        int port = int.Parse(_config["Smtp:Port"]); // Could throw at runtime
        string user = _config["Smtp:Username"];
    }
}
```

This approach has serious problems:

| Problem | Description |
|---|---|
| **Magic strings** | Typos like `"Smtp:Hots"` compile fine but fail silently at runtime |
| **No type safety** | Every value is a `string` -- you parse integers, booleans, and TimeSpans manually |
| **No IntelliSense** | Your IDE cannot autocomplete configuration keys |
| **No validation** | Missing or malformed values are only discovered at runtime, often deep in a call stack |
| **Tight coupling** | Every class depends on `IConfiguration`, making unit testing painful |
| **No grouping** | Related settings are scattered -- nothing enforces that "Smtp" settings travel together |

> [!danger] Critical Problem
> Injecting `IConfiguration` directly into every service is an anti-pattern. It is the service locator pattern applied to configuration -- your class hides its actual dependencies behind a grab bag of key-value pairs.

The Options pattern solves all of these problems by giving you a strongly-typed class that the framework populates, validates, and injects for you.

> [!summary] Section Summary
> Raw `IConfiguration` access scatters magic strings, lacks type safety, prevents IntelliSense, and couples every service to the entire configuration system. The Options pattern replaces this with strongly-typed POCOs.

---

## The POCO Class

A **POCO (Plain Old CLR Object)** class is the foundation of the Options pattern. Its public properties must match the keys in your configuration section.

### Configuration Source (appsettings.json)

```json
{
  "Smtp": {
    "Host": "smtp.example.com",
    "Port": 587,
    "Username": "noreply@example.com",
    "Password": "s3cret!",
    "EnableSsl": true,
    "Timeout": "00:00:30"
  }
}
```

### Options POCO Class

```csharp
public class SmtpSettings
{
    public const string SectionName = "Smtp";

    public string Host { get; set; } = string.Empty;
    public int Port { get; set; } = 587;
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
    public bool EnableSsl { get; set; } = true;
    public TimeSpan Timeout { get; set; } = TimeSpan.FromSeconds(30);
}
```

> [!info] Definition
> An **Options class** (also called an Options POCO) is a non-abstract class with a public parameterless constructor and public read-write properties. The property names must match the configuration keys (case-insensitive by default).

### Rules for Options Classes

1. The class **must** have a public parameterless constructor
2. Properties **must** be public with both `get` and `set` accessors
3. Property names are matched to configuration keys **case-insensitively**
4. Fields are **not** bound -- only properties
5. Nested objects and collections (lists, dictionaries) are supported
6. A `const string SectionName` is a common convention to avoid repeating the section name

> [!warning] Common Misconception
> The Options class does **not** need to be a record, and you should generally avoid using `record` types for Options classes. Records generate `init`-only setters by default, which the configuration binder cannot write to in older .NET versions. Use a plain class with `{ get; set; }` properties.

### Nested and Complex Types

Configuration binding supports nested objects and collections:

```json
{
  "AppFeatures": {
    "MaxRetries": 3,
    "AllowedOrigins": [ "https://app.example.com", "https://admin.example.com" ],
    "RateLimits": {
      "RequestsPerMinute": 100,
      "BurstSize": 20
    }
  }
}
```

```csharp
public class AppFeatureFlags
{
    public const string SectionName = "AppFeatures";

    public int MaxRetries { get; set; } = 3;
    public List<string> AllowedOrigins { get; set; } = new();
    public RateLimitOptions RateLimits { get; set; } = new();
}

public class RateLimitOptions
{
    public int RequestsPerMinute { get; set; } = 60;
    public int BurstSize { get; set; } = 10;
}
```

> [!summary] Section Summary
> Options POCOs are plain C# classes with public get/set properties matching configuration keys. They support default values, nested objects, and collections. Always define a `const string SectionName` for the configuration section name.

---

## Registering Options with DI

You register Options in `Program.cs` (or `Startup.ConfigureServices` in older projects) using the `Configure<T>` extension method on `IServiceCollection`.

### Basic Registration

```csharp
var builder = WebApplication.CreateBuilder(args);

// Bind the "Smtp" section to SmtpSettings
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection(SmtpSettings.SectionName));
```

### What Happens Under the Hood

When you call `services.Configure<SmtpSettings>(section)`:

1. The framework registers `IOptions<SmtpSettings>`, `IOptionsSnapshot<SmtpSettings>`, and `IOptionsMonitor<SmtpSettings>` in the DI container
2. When these interfaces are resolved, the framework reads the configuration section and binds it to a new `SmtpSettings` instance
3. All three interfaces are available from a single `Configure<T>` call

### Alternative: Bind and Get Immediately

Sometimes you need the options value in `Program.cs` itself (before the DI container is built):

```csharp
var smtpSection = builder.Configuration.GetSection(SmtpSettings.SectionName);
var smtpSettings = smtpSection.Get<SmtpSettings>();

// smtpSettings is a plain SmtpSettings object -- not wrapped in IOptions<T>
Console.WriteLine(smtpSettings?.Host);
```

> [!warning] Common Misconception
> `GetSection()` does **not** throw if the section is missing -- it returns an empty `IConfigurationSection`. Your Options class will be created with all default values. Use [[#Options Validation]] to catch missing sections at startup.

### Alternative: Bind Method

```csharp
var smtpSettings = new SmtpSettings();
builder.Configuration.GetSection(SmtpSettings.SectionName).Bind(smtpSettings);
```

### Registration with a Lambda (No Config Section)

You can also configure options purely in code:

```csharp
builder.Services.Configure<SmtpSettings>(options =>
{
    options.Host = "smtp.hardcoded.com";
    options.Port = 465;
    options.EnableSsl = true;
});
```

> [!tip] Pro Tip
> You can combine both approaches. The lambda runs **after** the configuration section binding, so you can override specific values:
> ```csharp
> builder.Services.Configure<SmtpSettings>(
>     builder.Configuration.GetSection("Smtp"));
> builder.Services.PostConfigure<SmtpSettings>(options =>
> {
>     // Override or set defaults after binding
>     options.Timeout = TimeSpan.FromSeconds(60);
> });
> ```

> [!summary] Section Summary
> Call `services.Configure<T>(config.GetSection("SectionName"))` to register options. This automatically registers `IOptions<T>`, `IOptionsSnapshot<T>`, and `IOptionsMonitor<T>`. Use `Get<T>()` or `Bind()` for one-off reads outside DI.

---

## Injecting Options into Services

Once registered, inject the options into your services using one of three interfaces.

### Basic Injection with IOptions

```csharp
public class EmailService
{
    private readonly SmtpSettings _settings;

    public EmailService(IOptions<SmtpSettings> options)
    {
        _settings = options.Value;
    }

    public async Task SendAsync(string to, string subject, string body)
    {
        using var client = new SmtpClient(_settings.Host, _settings.Port);
        client.EnableSsl = _settings.EnableSsl;
        client.Credentials = new NetworkCredential(
            _settings.Username, _settings.Password);

        // Full IntelliSense, type safety, no magic strings
        await client.SendMailAsync(
            new MailMessage(_settings.Username, to, subject, body));
    }
}
```

### Unit Testing with Options

One of the biggest wins: you can unit test without any configuration infrastructure.

```csharp
[Fact]
public async Task SendAsync_UsesConfiguredHost()
{
    // Arrange -- no appsettings.json, no IConfiguration, just a POCO
    var settings = new SmtpSettings
    {
        Host = "test-smtp.local",
        Port = 25,
        Username = "test@test.com",
        EnableSsl = false
    };

    var service = new EmailService(Options.Create(settings));

    // Act & Assert ...
}
```

> [!tip] Pro Tip
> `Options.Create<T>(value)` wraps any POCO in an `IOptions<T>` for testing. It lives in `Microsoft.Extensions.Options` and requires no mocking framework.

> [!summary] Section Summary
> Inject `IOptions<T>` into constructors and access `.Value` to get the POCO. For unit testing, use `Options.Create(new MySettings { ... })` to create test instances without configuration infrastructure.

---

## IOptions vs IOptionsSnapshot vs IOptionsMonitor

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

---

## Named Options

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

---

## Options Validation

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

---

## PostConfigure

**`PostConfigure<T>`** runs **after** all `Configure<T>` calls have executed. It is the last chance to set defaults, override values, or apply computed properties.

### Basic PostConfigure

```csharp
builder.Services.PostConfigure<SmtpSettings>(options =>
{
    // Set a default timeout if none was specified in configuration
    if (options.Timeout == TimeSpan.Zero)
        options.Timeout = TimeSpan.FromSeconds(30);

    // Force SSL in production
    if (!builder.Environment.IsDevelopment())
        options.EnableSsl = true;
});
```

### Execution Order

The configuration pipeline runs in this exact order:

1. **`Configure<T>(section)`** -- binds values from `IConfiguration`
2. **`Configure<T>(action)`** -- applies lambda overrides (in registration order)
3. **`PostConfigure<T>(action)`** -- runs after all `Configure` calls
4. **Validation** -- `ValidateDataAnnotations()`, `.Validate()`, `IValidateOptions<T>`

```csharp
// Step 1: Bind from appsettings.json
builder.Services.Configure<SmtpSettings>(
    builder.Configuration.GetSection("Smtp"));

// Step 2: Override in code
builder.Services.Configure<SmtpSettings>(o => o.Port = 587);

// Step 3: PostConfigure -- runs AFTER all Configure calls
builder.Services.PostConfigure<SmtpSettings>(o =>
    o.Timeout = o.Timeout == TimeSpan.Zero
        ? TimeSpan.FromSeconds(30)
        : o.Timeout);

// Step 4: Validation runs last
builder.Services.AddOptions<SmtpSettings>()
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

### Named PostConfigure

PostConfigure works with named options too:

```csharp
// Apply to a specific named option
builder.Services.PostConfigure<SmtpSettings>("Marketing", options =>
{
    options.EnableSsl = true;
});

// Apply to ALL named options (including the default)
builder.Services.PostConfigureAll<SmtpSettings>(options =>
{
    options.Timeout = TimeSpan.FromSeconds(30);
});
```

> [!warning] Common Misconception
> `PostConfigure` does **not** mean "after the app starts." It means "after all `Configure` registrations are applied to the options instance." It still runs at the time the options are first resolved (or at startup if `ValidateOnStart` is set).

> [!summary] Section Summary
> `PostConfigure<T>` runs after all `Configure<T>` calls, allowing you to set defaults and enforce invariants. Use `PostConfigureAll<T>` to apply defaults across all named instances. Validation runs after PostConfigure.

---

## Real-World Examples

### Example 1: JWT Authentication Settings

```json
{
  "Jwt": {
    "Issuer": "https://myapp.example.com",
    "Audience": "https://api.myapp.example.com",
    "SecretKey": "your-256-bit-secret-key-here-min-32-chars!",
    "ExpirationMinutes": 60,
    "RefreshExpirationDays": 7,
    "AllowedAlgorithms": [ "HS256", "HS384" ]
  }
}
```

```csharp
public class JwtSettings
{
    public const string SectionName = "Jwt";

    [Required]
    [Url]
    public string Issuer { get; set; } = string.Empty;

    [Required]
    [Url]
    public string Audience { get; set; } = string.Empty;

    [Required]
    [MinLength(32, ErrorMessage = "Secret key must be at least 32 characters for HS256")]
    public string SecretKey { get; set; } = string.Empty;

    [Range(1, 1440)]
    public int ExpirationMinutes { get; set; } = 60;

    [Range(1, 90)]
    public int RefreshExpirationDays { get; set; } = 7;

    public List<string> AllowedAlgorithms { get; set; } = new() { "HS256" };
}
```

```csharp
// Program.cs
builder.Services.AddOptions<JwtSettings>()
    .Bind(builder.Configuration.GetSection(JwtSettings.SectionName))
    .ValidateDataAnnotations()
    .Validate(jwt =>
    {
        if (jwt.RefreshExpirationDays * 24 * 60 <= jwt.ExpirationMinutes)
            return false;
        return true;
    }, "Refresh token expiration must be longer than access token expiration")
    .ValidateOnStart();
```

```csharp
// TokenService.cs
public class TokenService
{
    private readonly JwtSettings _jwt;

    public TokenService(IOptions<JwtSettings> options)
    {
        _jwt = options.Value;
    }

    public string GenerateToken(User user)
    {
        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_jwt.SecretKey));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var claims = new[]
        {
            new Claim(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
            new Claim(JwtRegisteredClaimNames.Email, user.Email),
            new Claim(ClaimTypes.Role, user.Role)
        };

        var token = new JwtSecurityToken(
            issuer: _jwt.Issuer,
            audience: _jwt.Audience,
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(_jwt.ExpirationMinutes),
            signingCredentials: credentials);

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

### Example 2: Feature Flags

```json
{
  "Features": {
    "EnableDarkMode": true,
    "EnableBetaApi": false,
    "MaxUploadSizeMb": 25,
    "MaintenanceWindow": {
      "Enabled": false,
      "StartUtc": "2026-06-20T02:00:00Z",
      "EndUtc": "2026-06-20T04:00:00Z"
    }
  }
}
```

```csharp
public class FeatureFlags
{
    public const string SectionName = "Features";

    public bool EnableDarkMode { get; set; }
    public bool EnableBetaApi { get; set; }

    [Range(1, 500)]
    public int MaxUploadSizeMb { get; set; } = 10;

    public MaintenanceWindowOptions MaintenanceWindow { get; set; } = new();
}

public class MaintenanceWindowOptions
{
    public bool Enabled { get; set; }
    public DateTime? StartUtc { get; set; }
    public DateTime? EndUtc { get; set; }
}
```

```csharp
// Use IOptionsMonitor for feature flags -- they may change at runtime
public class FeatureService
{
    private readonly IOptionsMonitor<FeatureFlags> _features;

    public FeatureService(IOptionsMonitor<FeatureFlags> features)
    {
        _features = features;

        _features.OnChange(flags =>
        {
            Console.WriteLine($"Feature flags updated. Dark mode: {flags.EnableDarkMode}");
        });
    }

    public bool IsDarkModeEnabled() => _features.CurrentValue.EnableDarkMode;

    public bool IsInMaintenanceWindow()
    {
        var mw = _features.CurrentValue.MaintenanceWindow;
        if (!mw.Enabled || mw.StartUtc is null || mw.EndUtc is null)
            return false;

        var now = DateTime.UtcNow;
        return now >= mw.StartUtc && now <= mw.EndUtc;
    }
}
```

> [!tip] Pro Tip
> Feature flags are a prime use case for `IOptionsMonitor<T>`. You can toggle features by editing `appsettings.json` (or an external config source) without restarting the application.

### Example 3: Multiple Database Connections (Named Options)

```json
{
  "Databases": {
    "Primary": {
      "ConnectionString": "Server=db1;Database=AppDb;...",
      "CommandTimeout": 30,
      "EnableRetry": true
    },
    "ReadReplica": {
      "ConnectionString": "Server=db2;Database=AppDb;...",
      "CommandTimeout": 15,
      "EnableRetry": false
    }
  }
}
```

```csharp
public class DatabaseOptions
{
    [Required]
    public string ConnectionString { get; set; } = string.Empty;

    [Range(5, 300)]
    public int CommandTimeout { get; set; } = 30;

    public bool EnableRetry { get; set; } = true;
}

public static class DatabaseOptionNames
{
    public const string Primary = nameof(Primary);
    public const string ReadReplica = nameof(ReadReplica);
}
```

```csharp
// Program.cs
builder.Services.Configure<DatabaseOptions>(
    DatabaseOptionNames.Primary,
    builder.Configuration.GetSection("Databases:Primary"));

builder.Services.Configure<DatabaseOptions>(
    DatabaseOptionNames.ReadReplica,
    builder.Configuration.GetSection("Databases:ReadReplica"));

// Ensure all named instances are validated
builder.Services.PostConfigureAll<DatabaseOptions>(opts =>
{
    if (opts.CommandTimeout < 5)
        opts.CommandTimeout = 5;
});
```

```csharp
// Repository using named options
public class OrderRepository
{
    private readonly DatabaseOptions _primary;
    private readonly DatabaseOptions _readReplica;

    public OrderRepository(IOptionsSnapshot<DatabaseOptions> options)
    {
        _primary = options.Get(DatabaseOptionNames.Primary);
        _readReplica = options.Get(DatabaseOptionNames.ReadReplica);
    }

    public async Task<Order> GetByIdAsync(int id)
    {
        // Use read replica for queries
        using var conn = new SqlConnection(_readReplica.ConnectionString);
        conn.Open();
        // ...
    }

    public async Task CreateAsync(Order order)
    {
        // Use primary for writes
        using var conn = new SqlConnection(_primary.ConnectionString);
        conn.Open();
        // ...
    }
}
```

> [!summary] Section Summary
> Real-world Options pattern usage spans JWT settings (validated at startup with `IOptions<T>`), feature flags (monitored at runtime with `IOptionsMonitor<T>`), and multi-database configurations (named options with `IOptionsSnapshot<T>`). Choose the injection interface based on your reload requirements.

---

## Comprehensive Summary

> [!tip] Complete Summary
> The **Options pattern** is the standard way to consume configuration in ASP.NET Core. Here is the complete mental model:
>
> **Core idea**: Bind configuration sections to strongly-typed POCOs with `services.Configure<T>(config.GetSection("Name"))`. This gives you type safety, IntelliSense, validation, and testability.
>
> **Three injection interfaces**:
> - `IOptions<T>` -- Singleton, reads once, simplest choice for static config
> - `IOptionsSnapshot<T>` -- Scoped, re-reads per request, cannot be used in Singletons
> - `IOptionsMonitor<T>` -- Singleton with live reload and `OnChange` callbacks, ideal for background services
>
> **Named options**: Register multiple configurations of the same type with `Configure<T>(name, section)` and access via `.Get(name)`.
>
> **Validation**: Use `ValidateDataAnnotations()`, `.Validate()`, or `IValidateOptions<T>` -- and **always** add `.ValidateOnStart()` to fail fast.
>
> **PostConfigure**: Runs after all `Configure` calls to set defaults or enforce invariants.
>
> **Testing**: Use `Options.Create(new MySettings { ... })` to create `IOptions<T>` wrappers without any configuration infrastructure.

---

## Related Topics

- [[Configuration Overview]] -- How the configuration system works end-to-end (providers, layering, precedence)
- [[Secrets and Environment Variables]] -- Managing sensitive settings outside of `appsettings.json`
- [[Strongly Typed Configuration]] -- Deeper dive into binding complex types, arrays, and dictionaries
- [[Dependency Injection]] -- The DI container that resolves `IOptions<T>` and its variants
- [[Middleware]] -- Where scoped options (`IOptionsSnapshot<T>`) get their per-request lifetime
- [[Background Services]] -- Primary consumer of `IOptionsMonitor<T>` for long-running tasks

---

## Further Reading

- [Microsoft Docs: Options pattern in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options)
- [Microsoft Docs: Configuration in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/)
- [Andrew Lock: Adding validation to strongly typed configuration objects](https://andrewlock.net/adding-validation-to-strongly-typed-configuration-objects-in-asp-net-core/)
- [Steve Smith: ASP.NET Core Options Pattern](https://ardalis.com/aspnetcore-options-pattern/)
