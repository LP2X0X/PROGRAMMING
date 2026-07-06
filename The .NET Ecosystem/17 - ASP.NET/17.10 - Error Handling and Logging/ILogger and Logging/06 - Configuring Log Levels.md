---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---


Log level filtering is configured in `appsettings.json` under the `"Logging"` section. You can set different minimum levels per category prefix.

## Default Configuration

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore": "Warning",
      "System": "Warning"
    }
  }
}
```

This configuration means:
- **All application code**: Information and above (Info, Warning, Error, Critical)
- **ASP.NET Core framework**: Warning and above (suppress the noisy Info logs from routing, hosting, etc.)
- **Entity Framework Core**: Warning and above (suppress the SQL query logs at Info level)
- **System namespace**: Warning and above

## Environment-Specific Configuration

Use `appsettings.Development.json` to configure more verbose logging during development:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Information",
      "Microsoft.EntityFrameworkCore.Database.Command": "Information",
      "MyApp": "Trace"
    }
  }
}
```

In development, this shows:
- Debug-level logs by default
- ASP.NET Core Information logs (request routing, middleware execution)
- EF Core SQL commands (the actual SQL queries being executed)
- Trace-level logs for your application code

## Per-Provider Configuration

You can also configure log levels per logging provider. This lets you send verbose logs to one destination and only errors to another:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    },
    "Console": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Debug": {
      "LogLevel": {
        "Default": "Debug"
      }
    }
  }
}
```

## Filtering Precedence

The filtering rules follow this precedence (first match wins):
1. Provider-specific category match (e.g., `"Console" > "MyApp.Services"`)
2. Provider-specific default (e.g., `"Console" > "Default"`)
3. Global category match (e.g., `"LogLevel" > "MyApp.Services"`)
4. Global default (e.g., `"LogLevel" > "Default"`)

> [!tip]
> When debugging a specific issue in production, you can temporarily lower the log level for just the relevant namespace without drowning in framework noise. For example, set `"MyApp.Services.PaymentService": "Debug"` while keeping everything else at Warning. If you use `appsettings.json` with `reloadOnChange: true` (the default), changes take effect without restarting the application.

> [!warning] Common Misconception
> Setting `"Default": "Trace"` in production is not "just more logs." Trace and Debug levels can generate thousands of log entries per second, overwhelming your log storage, degrading application performance, and potentially logging sensitive data that Trace-level code was never expected to hide. Always be deliberate about production log levels.

> [!summary] Section Summary
> - Configure log levels in `appsettings.json` under `"Logging"` > `"LogLevel"`
> - Use category prefixes for granular control: `"Microsoft.AspNetCore": "Warning"` suppresses noisy framework logs
> - Override per environment using `appsettings.Development.json` for verbose development logging
> - Per-provider configuration lets you send different verbosity levels to different destinations
> - Changes to `appsettings.json` take effect at runtime without restart (with `reloadOnChange: true`)
