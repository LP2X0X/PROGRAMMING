---
tags:
  - csharp
  - configuration
---

## Why Configuration Files?

Values that change between environments — connection strings, API keys, feature flags, timeouts — should not be hardcoded. Configuration files let you change application behavior **without recompiling**.

```ad-info
title: Key Principle
Separate *what can change* from *what is compiled*. Configuration is data, not code.
```

---

## Configuration File Types

| Format | File | Used By |
|---|---|---|
| JSON | `appsettings.json` | .NET Core / .NET 5+ (primary) |
| XML | `app.config` / `web.config` | .NET Framework (legacy) |
| Environment Variables | OS-level | Both |
| User Secrets | `secrets.json` | Development only |
| Command-line args | CLI flags | Both |

---

## Modern .NET (6+): appsettings.json

The **minimal hosting model** in .NET 6+ loads configuration automatically with zero boilerplate:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Configuration is automatically loaded from (in order):
// 1. appsettings.json
// 2. appsettings.{Environment}.json
// 3. User secrets (Development only)
// 4. Environment variables
// 5. Command-line arguments

var connString = builder.Configuration.GetConnectionString("Default");
```

A typical `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "Default": "Server=myserver;Database=mydb;Trusted_Connection=True;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "MaxRetryCount": 3
}
```

```ad-note
title: Build Action
Set the file's **Copy to Output Directory** to *Copy if newer* so it ships alongside your executable.
```

---

## .NET Framework (Legacy): app.config / web.config

```ad-warning
title: Legacy Pattern
This section covers .NET Framework only. For new projects, use `appsettings.json` with `IConfiguration`.
```

Desktop apps use `app.config`, web apps use `web.config` — both are XML:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <connectionStrings>
    <add name="Default"
         connectionString="Server=myserver;Database=mydb;Trusted_Connection=True;"
         providerName="System.Data.SqlClient" />
  </connectionStrings>
  <appSettings>
    <add key="MaxRetryCount" value="3" />
  </appSettings>
</configuration>
```

Access values with `ConfigurationManager` (requires `System.Configuration` NuGet package):

```csharp
using System.Configuration;

var connString = ConfigurationManager.ConnectionStrings["Default"].ConnectionString;
var retries = ConfigurationManager.AppSettings["MaxRetryCount"]; // returns string
```

---

## Accessing Configuration Values (.NET 6+)

`IConfiguration` provides several ways to read values:

```csharp
// Indexer — returns string or null
string value = builder.Configuration["MaxRetryCount"];

// GetSection — returns an IConfigurationSection for nested objects
var loggingSection = builder.Configuration.GetSection("Logging");

// GetConnectionString — shortcut for ConnectionStrings:{name}
var conn = builder.Configuration.GetConnectionString("Default");

// GetValue<T> — reads and converts a single key
int retries = builder.Configuration.GetValue<int>("MaxRetryCount");
```

```ad-warning
title: All Values Are Strings Internally
Configuration values are stored as `string`. `GetValue<T>` handles conversion, but the raw indexer always returns `string?`. Forgetting this causes subtle bugs with booleans and numbers.
```

---

## See Also

- [[Multiple Configuration Files]]
- [[Working with Objects in Configuration Files]]
- [[Bind and Get Methods]]
