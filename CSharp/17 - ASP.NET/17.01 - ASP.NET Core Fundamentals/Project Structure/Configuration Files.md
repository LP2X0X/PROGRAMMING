---
tags: [csharp, asp-net-core, project-structure]
---


ASP.NET Core uses a layered configuration system based on JSON files, environment variables, command-line arguments, and more. The two default configuration files are `appsettings.json` and `appsettings.{Environment}.json`.

### appsettings.json

This is the base configuration file loaded for all environments:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=InventoryDb;Trusted_Connection=true;"
  },
  "AppSettings": {
    "PageSize": 25,
    "MaxUploadSizeMB": 10,
    "SupportEmail": "support@example.com"
  }
}
```

### appsettings.Development.json

This file overrides values from `appsettings.json` when the application runs in the `Development` environment:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=InventoryDb_Dev;Trusted_Connection=true;"
  }
}
```

> [!ad-note] Configuration Layering
> Configuration sources are loaded in a specific order, and later sources override earlier ones:
> 1. `appsettings.json`
> 2. `appsettings.{Environment}.json`
> 3. User secrets (Development only)
> 4. Environment variables
> 5. Command-line arguments
>
> This means environment variables override JSON files, and command-line arguments override everything. See [[Environments]] for how the environment name is determined.

### Accessing Configuration in Code

```csharp
// Binding to a strongly-typed options class
public class AppSettings
{
    public int PageSize { get; set; }
    public int MaxUploadSizeMB { get; set; }
    public string SupportEmail { get; set; } = string.Empty;
}

// In Program.cs
builder.Services.Configure<AppSettings>(
    builder.Configuration.GetSection("AppSettings"));

// In a service or controller via DI
public class OrderService
{
    private readonly AppSettings _settings;

    public OrderService(IOptions<AppSettings> options)
    {
        _settings = options.Value;
    }
}
```

> [!warning] Never Store Secrets in appsettings.json
> Connection strings with passwords, API keys, and other sensitive data should never be committed to source control via `appsettings.json`. Use User Secrets during development (`dotnet user-secrets set`) and environment variables or a vault service (Azure Key Vault, AWS Secrets Manager) in production.

> [!summary] Section Summary
> - `appsettings.json` is the base configuration file; environment-specific files override it
> - Configuration is layered: JSON files, user secrets, environment variables, CLI arguments (in that priority order)
> - Use the Options pattern (`IOptions<T>`) to bind configuration sections to strongly-typed classes
> - Never store secrets in JSON config files committed to source control
