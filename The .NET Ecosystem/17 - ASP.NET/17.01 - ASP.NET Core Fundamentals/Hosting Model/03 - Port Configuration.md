---
tags: [csharp, asp-net-core, hosting, kestrel]
---


ASP.NET Core provides multiple ways to configure which ports and URLs the application listens on. They follow a clear precedence order.

### Precedence Order (Highest to Lowest)

1. **Code** -- `options.ListenAnyIP(5000)` in `ConfigureKestrel`
2. **Command-line argument** -- `--urls "http://0.0.0.0:5000"`
3. **Environment variable** -- `ASPNETCORE_URLS=http://0.0.0.0:5000`
4. **`appsettings.json`** -- Kestrel endpoint configuration
5. **`launchSettings.json`** -- Development only (not deployed)

### Command-Line `--urls`

```bash
dotnet run --urls "http://0.0.0.0:5000;https://0.0.0.0:5001"
```

### Environment Variable

```bash
# Linux/macOS
export ASPNETCORE_URLS="http://+:5000;https://+:5001"

# Windows PowerShell
$env:ASPNETCORE_URLS = "http://+:5000;https://+:5001"
```

### launchSettings.json (Development Only)

```json
{
  "profiles": {
    "OrderService": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "https://localhost:7201;http://localhost:5201",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

> [!ad-note] `launchSettings.json` Is Not Deployed
> The `launchSettings.json` file lives under `Properties/` and is only used during local development with `dotnet run` or IDE launch. It is not included in published output. For staging and production, use environment variables or `appsettings.{Environment}.json`.

### URL Format Reference

| Format | Meaning |
|---|---|
| `http://localhost:5000` | Loopback only (IPv4 + IPv6) |
| `http://0.0.0.0:5000` | All IPv4 interfaces |
| `http://[::]:5000` | All IPv6 interfaces |
| `http://+:5000` | All interfaces (shorthand) |
| `http://*:5000` | All interfaces (alternative shorthand) |
| `https://+:5001` | All interfaces with HTTPS |

> [!summary] Section Summary
> - Ports can be configured via code, CLI arguments, environment variables, or config files
> - `ASPNETCORE_URLS` is the most common method for production deployments
> - `launchSettings.json` is development-only and not included in published output
> - Use `+` or `*` to listen on all network interfaces; use `localhost` to restrict to loopback
