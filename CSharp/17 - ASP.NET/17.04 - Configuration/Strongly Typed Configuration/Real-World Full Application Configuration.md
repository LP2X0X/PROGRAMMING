---
tags:
  - csharp
  - asp-net-core
  - configuration
  - binding
  - strongly-typed
---


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
