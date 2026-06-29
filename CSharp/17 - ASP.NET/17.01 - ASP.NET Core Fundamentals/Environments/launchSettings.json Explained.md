---
tags: [csharp, asp-net-core, environments, configuration]
---


The `launchSettings.json` file configures how `dotnet run` and Visual Studio launch your application during development. It lives at `Properties/launchSettings.json` and is **not deployed to production**.

### Full Example

```json
{
  "$schema": "http://json.schemastore.org/launchsettings.json",
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:54321",
      "sslPort": 44321
    }
  },
  "profiles": {
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "http://localhost:5100",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "https://localhost:7100;http://localhost:5100",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

### Key Properties

| Property               | Description                                                                 |
|---|---|
| `commandName`          | How the app launches: `Project` (Kestrel), `IISExpress`, or `Executable`    |
| `applicationUrl`       | The URL(s) Kestrel listens on; separate multiple with semicolons            |
| `launchBrowser`        | Whether to open the browser automatically on launch                         |
| `environmentVariables` | Dictionary of environment variables set before the app starts               |
| `dotnetRunMessages`    | Whether `dotnet run` displays informational messages in the console         |

> [!ad-note] Profile Selection
> When you run `dotnet run`, the first profile with `"commandName": "Project"` is used by default. You can select a specific profile with `dotnet run --launch-profile "https"`. Visual Studio lets you pick the profile from a dropdown in the toolbar.

> [!warning] Do Not Deploy launchSettings.json
> This file is for local development only. It should not be included in your publish output. The default `.pubxml` and `dotnet publish` already exclude it, but be careful with custom deployment scripts.

> [!summary] Section Summary
> - `launchSettings.json` lives in `Properties/` and configures development-time launch behavior.
> - It defines profiles with application URLs, environment variables, and browser launch settings.
> - It is not deployed to production -- it only affects `dotnet run` and IDE launches.
> - The first `Project` profile is used by default unless you specify `--launch-profile`.
