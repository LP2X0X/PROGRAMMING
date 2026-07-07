---
tags: [csharp, asp-net-core, environments, configuration]
---


Both environment variables and command-line arguments can configure the application, but they serve different roles.

### Environment Variables

```bash
# Set the environment
export ASPNETCORE_ENVIRONMENT=Production

# Override configuration values using double-underscore separator
export ConnectionStrings__InventoryDb="Server=prod-server;Database=InventoryDb;..."
export EmailSettings__SmtpServer="smtp.production.com"
```

> [!ad-note] Double Underscore Convention
> In environment variables, the `__` (double underscore) replaces the `:` section separator used in JSON configuration. So `ConnectionStrings:InventoryDb` in JSON becomes `ConnectionStrings__InventoryDb` as an environment variable.

### Command-Line Arguments

```bash
dotnet run --environment Staging --urls "https://localhost:7200"
dotnet OrderService.dll --ConnectionStrings:InventoryDb="Server=staging-server;..."
```

### When to Use Each

| Approach              | Best For                                       |
|---|---|
| Environment variables | Server deployments, Docker containers, CI/CD   |
| Command-line args     | Quick local testing, overriding a single value  |
| launchSettings.json   | Standard local development workflow             |
| appsettings.*.json    | Structured, version-controlled configuration    |

> [!summary] Section Summary
> - Environment variables use `__` as a section separator and are ideal for server deployments.
> - Command-line arguments are convenient for quick local overrides.
> - Both override values from `appsettings.json` files, with command-line having the highest priority.
