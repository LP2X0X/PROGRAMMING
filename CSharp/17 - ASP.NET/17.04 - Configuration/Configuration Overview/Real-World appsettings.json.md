---
tags:
  - csharp
  - asp-net-core
  - configuration
  - fundamentals
---


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
