---
tags:
  - csharp
  - configuration
---

## How Multiple Configuration Sources Work

.NET's configuration system is built on **providers** that are additive. Each provider adds key-value pairs to a flat dictionary. When the same key exists in multiple providers, the **last one registered wins**.

---

## Default Provider Order (.NET 6+)

`WebApplication.CreateBuilder(args)` registers these providers in order:

1. `appsettings.json`
2. `appsettings.{Environment}.json` (e.g., `appsettings.Development.json`)
3. User secrets (`secrets.json`, Development environment only)
4. Environment variables
5. Command-line arguments

```ad-note
title: Last One Wins
If the same key appears in multiple providers, the later provider's value takes precedence. Command-line args override everything; `appsettings.json` is the baseline.
```

---

## Environment-Specific Overrides

**appsettings.json** (base — ships to all environments):

```json
{
  "ConnectionStrings": {
    "Default": "Server=prod-server;Database=mydb;Trusted_Connection=True;"
  }
}
```

**appsettings.Development.json** (overrides only in Development):

```json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=mydb_dev;Trusted_Connection=True;"
  }
}
```

When `DOTNET_ENVIRONMENT=Development`, the local connection string wins.

```ad-info
title: Environment Detection
The environment is set via the `DOTNET_ENVIRONMENT` (or `ASPNETCORE_ENVIRONMENT` for web apps) variable. Common values: `Development`, `Staging`, `Production`. If unset, defaults to `Production`.
```

---

## Adding Custom Configuration Files

Register additional JSON files after the builder is created:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Configuration.AddJsonFile(
    "custom-settings.json",
    optional: true,        // don't throw if file is missing
    reloadOnChange: true); // pick up edits without restart
```

Custom files added this way are appended **after** the defaults, so their values override `appsettings.json` and environment-specific files.

```ad-warning
title: Order Matters
If you need a custom file to be the baseline (overridden by environment files), you must clear the default sources and re-add them in your desired order. In practice, this is rarely needed.
```

---

## See Also

- [[Configuring Applications with Configuration Files]]
- [[Working with Objects in Configuration Files]]
