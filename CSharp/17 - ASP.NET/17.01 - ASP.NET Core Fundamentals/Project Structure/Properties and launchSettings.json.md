---
tags: [csharp, asp-net-core, project-structure]
---


The `Properties/launchSettings.json` file configures how the application launches during local development. It defines profiles that specify URLs, environment variables, and launch behavior.

```json
{
  "$schema": "http://json.schemastore.org/launchsettings.json",
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:29085",
      "sslPort": 44340
    }
  },
  "profiles": {
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "http://localhost:5128",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "https://localhost:7205;http://localhost:5128",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "launchUrl": "swagger",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

### Key Properties

| Property | Purpose |
|---|---|
| `commandName` | `Project` runs via Kestrel; `IISExpress` uses IIS Express |
| `applicationUrl` | The URL(s) the app listens on; separate multiple with semicolons |
| `launchBrowser` | Whether to open the browser automatically on `dotnet run` |
| `launchUrl` | The relative URL to open (e.g., `swagger` for API projects) |
| `environmentVariables` | Sets environment variables; `ASPNETCORE_ENVIRONMENT` controls the active environment |

> [!tip] Selecting a Launch Profile
> Run a specific profile with `dotnet run --launch-profile https`. If you omit the flag, the first profile in the file is used.

> [!ad-note] Development Only
> `launchSettings.json` is only used during local development. It is not deployed to production. In production, URLs and environment variables are configured through hosting infrastructure (reverse proxy, container orchestration, etc.). See [[Hosting Model]] for details.

> [!summary] Section Summary
> - `launchSettings.json` lives in `Properties/` and controls local development launch behavior
> - Profiles define URLs, environment variables, and browser launch settings
> - `ASPNETCORE_ENVIRONMENT` in this file sets the active environment (typically `Development`)
> - This file is not used in production deployments
