---
tags: [csharp, asp-net-core, environments, configuration]
---


ASP.NET Core's configuration system automatically loads environment-specific JSON files that override the base `appsettings.json`.

### Loading Order

The configuration system loads files in this order (later files override earlier ones):

1. `appsettings.json` -- base settings shared across all environments
2. `appsettings.{Environment}.json` -- environment-specific overrides
3. User Secrets (Development only)
4. Environment variables
5. Command-line arguments

### Example: Base Configuration

`appsettings.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "InventoryDb": "Server=prod-db-server;Database=InventoryDb;Trusted_Connection=True;"
  },
  "EmailSettings": {
    "SmtpServer": "smtp.company.com",
    "Port": 587,
    "EnableSsl": true
  }
}
```

### Example: Development Override

`appsettings.Development.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Information"
    }
  },
  "ConnectionStrings": {
    "InventoryDb": "Server=localhost;Database=InventoryDb_Dev;Trusted_Connection=True;"
  },
  "EmailSettings": {
    "SmtpServer": "localhost",
    "Port": 1025,
    "EnableSsl": false
  }
}
```

### Example: Staging Override

`appsettings.Staging.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "InventoryDb": "Server=staging-db-server;Database=InventoryDb_Staging;Trusted_Connection=True;"
  }
}
```

> [!tip] Partial Overrides
> Environment-specific files do not need to repeat every setting from the base file. Only include the keys you want to override. All other values fall through from `appsettings.json`.

> [!warning] Never Store Secrets in appsettings.json
> Connection strings with passwords, API keys, and other secrets should use User Secrets (in Development) or environment variables / Azure Key Vault / secret managers (in Staging and Production). The `appsettings.*.json` files are checked into source control.

### How It Works in Program.cs

The `WebApplication.CreateBuilder(args)` call sets this up automatically:

```csharp
var builder = WebApplication.CreateBuilder(args);

// This is already done for you internally:
// builder.Configuration
//     .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
//     .AddJsonFile($"appsettings.{builder.Environment.EnvironmentName}.json", optional: true, reloadOnChange: true)
//     .AddUserSecrets<Program>(optional: true)  // Development only
//     .AddEnvironmentVariables()
//     .AddCommandLine(args);
```

> [!summary] Section Summary
> - `appsettings.json` provides base configuration for all environments.
> - `appsettings.{Environment}.json` overrides specific keys for that environment.
> - The override is automatic -- no extra code needed in `Program.cs`.
> - Later sources (environment variables, command-line) override earlier ones.
> - Never store secrets in JSON files that are committed to source control.
