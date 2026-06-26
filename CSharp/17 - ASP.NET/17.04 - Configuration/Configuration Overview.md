---
tags: [csharp, asp-net-core, configuration, fundamentals]
date: 2026-06-18
aliases: [Configuration System, ASP.NET Core Configuration, IConfiguration]
status: complete
---

# Configuration Overview

> [!abstract] Overview
> ASP.NET Core replaces the monolithic `web.config` / `ConfigurationManager` approach with a modern, **layered, provider-based configuration system**. Multiple sources (JSON files, environment variables, command-line arguments, user secrets, and more) are stacked in a defined order where **later sources override earlier ones**. The entire system is built around the `IConfiguration` interface and integrates seamlessly with dependency injection, the Options pattern, and environment-based overrides.

---

## Table of Contents

- [[#What Is the Configuration System]]
- [[#Configuration Providers and Load Order]]
- [[#How the Default Builder Sets Everything Up]]
- [[#IConfiguration Interface]]
- [[#Reading Configuration Values]]
- [[#Reading Sections]]
- [[#Connection Strings]]
- [[#Adding Custom Providers]]
- [[#IConfiguration vs IConfigurationRoot]]
- [[#Real-World appsettings.json]]
- [[#Comparison to web.config and ConfigurationManager]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]
- [[#Further Reading]]

---

## What Is the Configuration System

> [!info] Definition
> The **configuration system** in ASP.NET Core is a unified, extensible framework for reading application settings from multiple sources at startup and making them available throughout the application via dependency injection.

The core ideas are:

- **Provider-based** -- each source of configuration (JSON file, environment variable, etc.) has a corresponding **configuration provider** that knows how to read key-value pairs from that source.
- **Layered / override semantics** -- providers are registered in a specific order. When two providers supply the same key, the **last one registered wins**.
- **Flat key space with hierarchy** -- all configuration is normalized into a flat dictionary of `string` keys and `string` values. Hierarchical structure is expressed with a colon delimiter (e.g., `Smtp:Host`).
- **No special XML format** -- unlike the old `web.config`, configuration files are plain JSON (by default), and the system is format-agnostic.

```
appsettings.json          -->  base settings
appsettings.Production.json --> environment override
Environment variables     -->  deployment override
Command-line args         -->  one-off override
```

Each layer can override any key from the layers below it without modifying those files.

> [!summary] Section Summary
> ASP.NET Core configuration is a layered, provider-based system that replaces `web.config`. Multiple sources are stacked so that later sources override earlier ones, giving you flexible, environment-aware settings with no XML ceremony.

---

## Configuration Providers and Load Order

**Configuration providers** are the pluggable components that read settings from a source and feed them into the configuration system. `WebApplication.CreateBuilder()` registers the following providers **in this default order**:

| Order | Provider | Source | Override Scope |
|-------|----------|--------|----------------|
| 1 | `JsonConfigurationProvider` | `appsettings.json` | Base / shared settings |
| 2 | `JsonConfigurationProvider` | `appsettings.{Environment}.json` | Per-environment overrides |
| 3 | `UserSecretsConfigurationProvider` | `secrets.json` (local machine) | Development-only secrets |
| 4 | `EnvironmentVariablesConfigurationProvider` | OS environment variables | Deployment / container config |
| 5 | `CommandLineConfigurationProvider` | CLI arguments | One-off runtime overrides |

> [!warning] Override Rule
> The override rule is simple but critical: **the provider registered last wins**. If `appsettings.json` sets `MaxRetries` to `3` and an environment variable sets it to `5`, the value is `5` because environment variables are loaded after JSON files.

### Provider Details

### 1. appsettings.json

The base JSON file that ships with every project. Contains default values suitable for all environments.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  },
  "MaxRetries": 3
}
```

### 2. appsettings.{Environment}.json

An environment-specific file (e.g., `appsettings.Development.json`, `appsettings.Production.json`). The environment name comes from the `ASPNETCORE_ENVIRONMENT` variable.

```json
// appsettings.Development.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  },
  "MaxRetries": 10
}
```

> [!tip] File Is Optional
> Environment-specific JSON files are loaded with `optional: true`. If `appsettings.Staging.json` does not exist, the app starts normally using the base `appsettings.json` values.

### 3. User Secrets (Development Only)

**User secrets** store sensitive values (API keys, connection strings) outside the project directory so they are never committed to source control. They are only loaded when the environment is `Development`.

```bash
dotnet user-secrets init
dotnet user-secrets set "Smtp:Password" "my-secret-password"
```

Secrets are stored in a per-user location:
- Windows: `%APPDATA%\Microsoft\UserSecrets\<user_secrets_id>\secrets.json`
- Linux/macOS: `~/.microsoft/usersecrets/<user_secrets_id>/secrets.json`

> [!danger] Never Commit Secrets
> User secrets are a **development-only** convenience. In production, use environment variables, Azure Key Vault, AWS Secrets Manager, or another secure store. Never put real credentials in `appsettings.json`.

### 4. Environment Variables

Environment variables are the standard mechanism for injecting configuration in containers, CI/CD pipelines, and cloud platforms. Hierarchical keys use a **double underscore** (`__`) as the delimiter instead of a colon because colons are not valid in environment variable names on all platforms.

```bash
# Sets the key "Smtp:Host" in configuration
export Smtp__Host=smtp.example.com

# Sets the key "ConnectionStrings:Default"
export ConnectionStrings__Default="Server=prod-db;Database=App;..."
```

> [!tip] Prefix Filtering
> You can scope environment variables with a prefix to avoid conflicts:
> ```csharp
> builder.Configuration.AddEnvironmentVariables(prefix: "MYAPP_");
> ```
> Only variables starting with `MYAPP_` are loaded, and the prefix is stripped from the key.

### 5. Command-Line Arguments

The highest-priority default provider. Useful for one-off overrides when launching the app.

```bash
dotnet run --MaxRetries 20 --Smtp:Host=smtp.override.com
```

Supported formats:
- `--key value`
- `--key=value`
- `/key value`
- `/key=value`

> [!summary] Section Summary
> Five default providers are loaded in a fixed order: `appsettings.json`, `appsettings.{Environment}.json`, user secrets (Development only), environment variables, and command-line arguments. Later providers override earlier ones, so command-line arguments have the highest priority.

---

## How the Default Builder Sets Everything Up

When you call `WebApplication.CreateBuilder(args)`, the framework configures the entire configuration pipeline automatically. You do not need to manually register any of the default providers.

```csharp
var builder = WebApplication.CreateBuilder(args);

// At this point, builder.Configuration already contains values from:
// 1. appsettings.json
// 2. appsettings.{Environment}.json
// 3. User secrets (if Development)
// 4. Environment variables
// 5. Command-line arguments (from 'args')

var app = builder.Build();
```

Under the hood, `CreateBuilder` does roughly this:

```csharp
// Pseudocode of what CreateBuilder sets up
var config = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, reloadOnChange: true)
    .AddUserSecrets<Program>(optional: true)       // Development only
    .AddEnvironmentVariables()
    .AddCommandLine(args)
    .Build();
```

> [!ad-note] reloadOnChange
> The JSON providers use `reloadOnChange: true` by default. This means if you edit `appsettings.json` while the app is running, the configuration system detects the change and reloads values. This works with `IOptionsMonitor<T>` but **not** with values you read once at startup and cache.

> [!summary] Section Summary
> `WebApplication.CreateBuilder(args)` sets up all five default configuration providers automatically. You get a fully configured `IConfiguration` instance without writing any provider registration code.

---

## IConfiguration Interface

> [!info] Definition
> **`IConfiguration`** is the primary interface for reading configuration values. It represents a set of key-value pairs and supports hierarchical access through sections.

`IConfiguration` is registered in the DI container automatically and can be injected into any class:

```csharp
public class EmailService
{
    private readonly IConfiguration _config;

    public EmailService(IConfiguration config)
    {
        _config = config;
    }

    public void SendEmail()
    {
        string host = _config["Smtp:Host"];
        int port = _config.GetValue<int>("Smtp:Port");
        // ...
    }
}
```

### Key Members of IConfiguration

| Member | Description |
|--------|-------------|
| `this[string key]` | Indexer -- returns the value as `string?` or `null` if not found |
| `GetValue<T>(string key)` | Returns the value converted to type `T` |
| `GetValue<T>(string key, T defaultValue)` | Returns the value or a default if missing |
| `GetSection(string key)` | Returns an `IConfigurationSection` for a nested group |
| `GetChildren()` | Returns all immediate child sections |
| `GetConnectionString(string name)` | Shortcut for `this[$"ConnectionStrings:{name}"]` |

> [!warning] Null Return, Not Exception
> The indexer `config["Key"]` returns `null` if the key does not exist. It does **not** throw. Always handle the possibility of missing keys, or use `GetValue<T>` with a default.

> [!summary] Section Summary
> `IConfiguration` is the central interface for reading config values. It supports indexer access, typed retrieval via `GetValue<T>`, hierarchical navigation via `GetSection`, and is automatically available through dependency injection.

---

## Reading Configuration Values

### Indexer Access

The simplest way to read a value. Always returns `string?`.

```csharp
// Flat key
string? appName = config["AppName"];

// Hierarchical key (colon-delimited)
string? smtpHost = config["Smtp:Host"];

// Deeply nested
string? defaultLogLevel = config["Logging:LogLevel:Default"];
```

### Typed Access with GetValue

`GetValue<T>` converts the string value to the specified type using `TypeConverter`.

```csharp
int maxRetries = config.GetValue<int>("MaxRetries");
bool enableCache = config.GetValue<bool>("FeatureFlags:EnableCache");
TimeSpan timeout = config.GetValue<TimeSpan>("RequestTimeout");
```

> [!tip] Always Provide a Default
> `GetValue<T>(key)` returns `default(T)` if the key is missing -- which is `0` for `int`, `false` for `bool`, etc. This can mask misconfiguration. Prefer the overload with an explicit default:
> ```csharp
> int maxRetries = config.GetValue<int>("MaxRetries", defaultValue: 3);
> ```

### Example: Reading Multiple Values

Given this `appsettings.json`:

```json
{
  "AppName": "OrderProcessor",
  "MaxRetries": 5,
  "Smtp": {
    "Host": "smtp.example.com",
    "Port": 587,
    "UseSsl": true
  }
}
```

```csharp
string? appName = config["AppName"];               // "OrderProcessor"
int maxRetries = config.GetValue<int>("MaxRetries"); // 5
string? host = config["Smtp:Host"];                  // "smtp.example.com"
int port = config.GetValue<int>("Smtp:Port");        // 587
bool useSsl = config.GetValue<bool>("Smtp:UseSsl");  // true
```

> [!summary] Section Summary
> Read values with the indexer (`config["Key"]`) for strings, or `GetValue<T>` for typed conversion. Use the colon (`:`) delimiter for hierarchical keys. Always consider providing a default value to avoid silent failures.

---

## Reading Sections

**`GetSection`** returns an `IConfigurationSection` representing a subtree of the configuration. This is useful for grouping related settings.

```csharp
IConfigurationSection smtpSection = config.GetSection("Smtp");

string? host = smtpSection["Host"];        // No need for "Smtp:Host"
int port = smtpSection.GetValue<int>("Port");
```

### Checking If a Section Exists

`GetSection` never returns `null`. To check whether a section actually has values, use the `Exists()` extension method:

```csharp
IConfigurationSection section = config.GetSection("FeatureFlags");

if (section.Exists())
{
    // Section has values
}
```

### Enumerating Children

```csharp
IConfigurationSection loggingSection = config.GetSection("Logging:LogLevel");

foreach (IConfigurationSection child in loggingSection.GetChildren())
{
    Console.WriteLine($"{child.Key} = {child.Value}");
}
// Output:
// Default = Information
// Microsoft = Warning
// Microsoft.Hosting.Lifetime = Information
```

### Binding a Section to a POCO

Sections can be bound to a plain C# object, which is the foundation for the [[Options Pattern]]:

```csharp
public class SmtpSettings
{
    public string Host { get; set; } = string.Empty;
    public int Port { get; set; }
    public bool UseSsl { get; set; }
}
```

```csharp
var smtpSettings = new SmtpSettings();
config.GetSection("Smtp").Bind(smtpSettings);

// Or the one-liner equivalent:
SmtpSettings smtp = config.GetSection("Smtp").Get<SmtpSettings>()!;
```

> [!warning] Bind Does Not Throw on Missing Keys
> If a property in the POCO has no matching configuration key, it retains its default value. The binding silently ignores missing keys -- it does **not** throw. Validate your objects after binding if certain properties are required.

> [!summary] Section Summary
> `GetSection` provides scoped access to a subtree of configuration. Sections can be enumerated with `GetChildren()`, checked with `Exists()`, and bound to POCOs with `Bind()` or `Get<T>()` for strongly typed access.

---

## Connection Strings

Connection strings are so commonly needed that the configuration system provides a dedicated shortcut.

```csharp
string? connStr = config.GetConnectionString("Default");

// This is exactly equivalent to:
string? connStr = config["ConnectionStrings:Default"];
```

The corresponding JSON structure:

```json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=MyApp;Trusted_Connection=True;",
    "Reporting": "Server=reporting-db;Database=Reports;User=sa;Password=..."
  }
}
```

### Using Connection Strings in DI Registration

```csharp
var connectionString = builder.Configuration.GetConnectionString("Default")
    ?? throw new InvalidOperationException("Connection string 'Default' not found.");

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
```

> [!tip] Throw on Missing Connection Strings
> Unlike general config values, a missing connection string almost always means the app cannot function. Fail fast with an explicit exception rather than passing `null` to your database provider.

> [!summary] Section Summary
> `GetConnectionString("Name")` is a convenience shortcut for `config["ConnectionStrings:Name"]`. Always validate that the connection string is not null before passing it to database providers.

---

## Adding Custom Providers

You can extend the default configuration by adding extra providers to `builder.Configuration`.

### Adding a Custom JSON File

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Configuration.AddJsonFile("custom-settings.json", optional: true, reloadOnChange: true);
```

> [!warning] Order Matters
> Custom providers added after `CreateBuilder` are registered **after** the defaults. This means values in `custom-settings.json` will override values from environment variables and command-line arguments -- which is usually not what you want. If you need the custom file to be low-priority, clear and rebuild:
> ```csharp
> builder.Configuration.Sources.Clear();
> builder.Configuration
>     .AddJsonFile("custom-settings.json", optional: true)
>     .AddJsonFile("appsettings.json", optional: false)
>     .AddJsonFile($"appsettings.{builder.Environment.EnvironmentName}.json", optional: true)
>     .AddEnvironmentVariables()
>     .AddCommandLine(args);
> ```

### Adding an In-Memory Provider

Useful for testing:

```csharp
builder.Configuration.AddInMemoryCollection(new Dictionary<string, string?>
{
    ["Smtp:Host"] = "localhost",
    ["Smtp:Port"] = "25",
    ["MaxRetries"] = "1"
});
```

### Adding an XML File

```csharp
builder.Configuration.AddXmlFile("legacy-config.xml", optional: true);
```

### Adding an INI File

```csharp
builder.Configuration.AddIniFile("app.ini", optional: true);
```

### Other Built-In Providers

| Provider | NuGet Package | Use Case |
|----------|--------------|----------|
| Azure Key Vault | `Azure.Extensions.AspNetCore.Configuration.Secrets` | Production secrets in Azure |
| AWS Systems Manager | `Amazon.Extensions.Configuration.SystemsManager` | AWS parameter store |
| Azure App Configuration | `Microsoft.Extensions.Configuration.AzureAppConfiguration` | Centralized config in Azure |
| Custom provider | Implement `IConfigurationSource` + `IConfigurationProvider` | Any custom source |

> [!summary] Section Summary
> Custom providers can be added via `builder.Configuration.AddXxxFile()` or `AddInMemoryCollection()`. Be aware that providers added after `CreateBuilder` have higher priority than the defaults. Reorder sources explicitly if you need custom priority.

---

## IConfiguration vs IConfigurationRoot

> [!info] Definition
> **`IConfigurationRoot`** extends `IConfiguration` with additional capabilities: it exposes the list of all registered providers and supports explicit reload. It is the concrete type returned by `ConfigurationBuilder.Build()`.

| Feature | `IConfiguration` | `IConfigurationRoot` |
|---------|------------------|----------------------|
| Read values | Yes | Yes |
| Get sections | Yes | Yes |
| List providers | No | Yes (`.Providers`) |
| Force reload | No | Yes (`.Reload()`) |
| Typical usage | Inject into services | Available at startup |

### When to Use Each

- **`IConfiguration`** -- inject this into your services. It is the abstraction your code should depend on.
- **`IConfigurationRoot`** -- use at startup when you need to inspect providers or force a reload. Do not inject it into services unless you have a specific reason.

```csharp
var builder = WebApplication.CreateBuilder(args);

// builder.Configuration is IConfigurationManager (which implements IConfigurationRoot)
IConfigurationRoot configRoot = (IConfigurationRoot)builder.Configuration;

// Inspect registered providers
foreach (IConfigurationProvider provider in configRoot.Providers)
{
    Console.WriteLine(provider.ToString());
}
```

> [!tip] Debugging Configuration Sources
> Casting to `IConfigurationRoot` and iterating `.Providers` is invaluable when troubleshooting why a value is not what you expect. You can see exactly which providers are registered and in what order.

### GetDebugView

`IConfigurationRoot` has a `GetDebugView()` extension method that dumps every key-value pair along with which provider supplied it:

```csharp
Console.WriteLine(((IConfigurationRoot)builder.Configuration).GetDebugView());
```

Output:

```
AllowedHosts=* (JsonConfigurationProvider for 'appsettings.json' (Optional))
ConnectionStrings:
  Default=Server=localhost;... (JsonConfigurationProvider for 'appsettings.Development.json' (Optional))
Logging:
  LogLevel:
    Default=Debug (JsonConfigurationProvider for 'appsettings.Development.json' (Optional))
    Microsoft=Warning (JsonConfigurationProvider for 'appsettings.json' (Optional))
```

> [!summary] Section Summary
> `IConfigurationRoot` extends `IConfiguration` with provider inspection and manual reload. Use `IConfiguration` in services; use `IConfigurationRoot` at startup for debugging. `GetDebugView()` is a powerful diagnostic tool.

---

## Real-World appsettings.json

Here is a realistic `appsettings.json` for a typical web application:

```json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=MyApp;Trusted_Connection=True;TrustServerCertificate=True;",
    "Redis": "localhost:6379"
  },

  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore": "Warning"
    }
  },

  "AllowedHosts": "*",

  "Smtp": {
    "Host": "smtp.example.com",
    "Port": 587,
    "UseSsl": true,
    "SenderName": "MyApp Notifications",
    "SenderEmail": "noreply@example.com"
  },

  "Jwt": {
    "Issuer": "https://myapp.example.com",
    "Audience": "https://myapp.example.com",
    "ExpiryMinutes": 60
  },

  "AppSettings": {
    "PageSize": 25,
    "MaxUploadSizeMb": 10,
    "EnableMaintenanceMode": false
  },

  "ExternalApis": {
    "WeatherService": {
      "BaseUrl": "https://api.weather.example.com/v2",
      "TimeoutSeconds": 30
    }
  }
}
```

And the corresponding `appsettings.Development.json` that overrides only what differs:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.EntityFrameworkCore": "Information"
    }
  },

  "Smtp": {
    "Host": "localhost",
    "Port": 25,
    "UseSsl": false
  },

  "AppSettings": {
    "EnableMaintenanceMode": false
  }
}
```

> [!example] How Override Works in Practice
> In Development, the final resolved values for SMTP are:
> - `Smtp:Host` = `"localhost"` (overridden by Development file)
> - `Smtp:Port` = `25` (overridden)
> - `Smtp:UseSsl` = `false` (overridden)
> - `Smtp:SenderName` = `"MyApp Notifications"` (kept from base file)
> - `Smtp:SenderEmail` = `"noreply@example.com"` (kept from base file)
>
> Only the keys present in the environment file are overridden. Keys not mentioned in the override file retain their base values.

> [!danger] Sensitive Values
> Never put passwords, API keys, or secrets in `appsettings.json`. Use user secrets in Development and environment variables or a vault in Production. The JSON files are committed to source control and visible to everyone with repo access.

> [!summary] Section Summary
> A real-world `appsettings.json` contains connection strings, logging configuration, feature settings, and external API endpoints. Environment-specific files override only the keys that differ, keeping the base file as the single source of truth for defaults.

---

## Comparison to web.config and ConfigurationManager

The legacy .NET Framework approach used `web.config` (or `app.config`) with `ConfigurationManager`. ASP.NET Core replaces this entirely.

| Aspect | Legacy (`web.config`) | Modern (`IConfiguration`) |
|--------|----------------------|--------------------------|
| Format | XML | JSON (default), XML, INI, or any custom format |
| Access | `ConfigurationManager.AppSettings["Key"]` | `config["Key"]` or `config.GetValue<T>("Key")` |
| Connection strings | `ConfigurationManager.ConnectionStrings["Name"].ConnectionString` | `config.GetConnectionString("Name")` |
| Hierarchy | Flat `<appSettings>` or custom sections with boilerplate | Native hierarchy with colon delimiter |
| Environment overrides | Web.config transforms (SlowCheetah, XSLT) | Automatic with `appsettings.{Env}.json` |
| Secrets management | Encrypted sections in XML (complex) | User secrets, environment variables, vaults |
| Reload on change | Requires app restart | Built-in with `reloadOnChange: true` |
| Multiple sources | Complex, manual merging | First-class layered provider model |
| Strongly typed | Manual parsing or custom `ConfigurationSection` classes | [[Options Pattern]] with `IOptions<T>` |
| Testability | Global static -- hard to mock | Interface-based -- easy to mock |
| DI integration | None (static access) | Fully integrated |

### Legacy Code Example

```csharp
// .NET Framework -- web.config
// <appSettings>
//   <add key="MaxRetries" value="3" />
// </appSettings>

string maxRetriesStr = ConfigurationManager.AppSettings["MaxRetries"];
int maxRetries = int.Parse(maxRetriesStr);

string connStr = ConfigurationManager.ConnectionStrings["Default"].ConnectionString;
```

### Modern Equivalent

```csharp
// ASP.NET Core -- appsettings.json
// { "MaxRetries": 3, "ConnectionStrings": { "Default": "..." } }

int maxRetries = config.GetValue<int>("MaxRetries");
string? connStr = config.GetConnectionString("Default");
```

> [!warning] ConfigurationManager Still Exists in .NET 6+
> The `System.Configuration.ConfigurationManager` class was ported to .NET 6+ for backward compatibility. Do **not** use it in new ASP.NET Core projects. It reads from `app.config` (not `appsettings.json`) and does not participate in the modern configuration pipeline.

> [!summary] Section Summary
> The modern configuration system is a significant improvement over `web.config`: JSON by default, hierarchical, layered, testable, DI-friendly, and supports runtime reload. The legacy `ConfigurationManager` is a static, XML-based, flat system that should not be used in new ASP.NET Core applications.

---

## Comprehensive Summary

> [!tip] Complete Summary
> **The ASP.NET Core configuration system** is a provider-based, layered architecture where multiple sources contribute key-value pairs and later sources override earlier ones.
>
> **Default provider order** (low to high priority):
> 1. `appsettings.json` -- base defaults
> 2. `appsettings.{Environment}.json` -- environment-specific overrides
> 3. User secrets -- development-only sensitive values
> 4. Environment variables -- deployment/container config
> 5. Command-line arguments -- highest priority, one-off overrides
>
> **Key interfaces:**
> - `IConfiguration` -- the primary abstraction for reading values; inject this into services
> - `IConfigurationRoot` -- extends `IConfiguration` with provider inspection and reload; use at startup for diagnostics
>
> **Reading values:**
> - Indexer: `config["Section:Key"]` returns `string?`
> - Typed: `config.GetValue<int>("Key", defaultValue)` with type conversion
> - Sections: `config.GetSection("Group")` for scoped access
> - Connection strings: `config.GetConnectionString("Name")` as a shortcut
>
> **Best practices:**
> - Never commit secrets to JSON files
> - Use `GetDebugView()` to troubleshoot configuration resolution
> - Prefer the [[Options Pattern]] for strongly typed settings
> - Be aware of provider order when adding custom sources
> - Fail fast on missing critical configuration (connection strings, required API keys)

---

## Related Topics

- [[Options Pattern]] -- strongly typed configuration with `IOptions<T>`, `IOptionsSnapshot<T>`, and `IOptionsMonitor<T>`
- [[Secrets and Environment Variables]] -- secure management of sensitive configuration values
- [[Strongly Typed Configuration]] -- binding configuration sections to POCO classes
- [[Environments]] -- how `ASPNETCORE_ENVIRONMENT` drives environment-specific behavior
- [[Dependency Injection]] -- how `IConfiguration` is registered and resolved in the DI container

---

## Further Reading

- Microsoft Docs: Configuration in ASP.NET Core -- https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration
- Microsoft Docs: Safe storage of app secrets in development -- https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets
- Microsoft Docs: Options pattern in ASP.NET Core -- https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options
- Microsoft Docs: Use multiple environments -- https://learn.microsoft.com/en-us/aspnet/core/fundamentals/environments
- Andrew Lock: Exploring the .NET Core Configuration system -- https://andrewlock.net/exploring-the-microsoft-extensions-configuration-libraries
