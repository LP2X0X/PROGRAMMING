---
tags: [csharp, asp-net-core, environments, configuration]
---


The environment is controlled by the `ASPNETCORE_ENVIRONMENT` environment variable. There are several places you can set it.

### Setting via Environment Variable

On Windows (PowerShell):

```powershell
$env:ASPNETCORE_ENVIRONMENT = "Development"
dotnet run
```

On Windows (Command Prompt):

```bash
set ASPNETCORE_ENVIRONMENT=Development
dotnet run
```

On Linux/macOS:

```bash
export ASPNETCORE_ENVIRONMENT=Development
dotnet run
```

### Setting via launchSettings.json (Development Only)

The most common way during local development is through `launchSettings.json`, which lives in the `Properties` folder of your project.

### Setting via Command-Line Argument

```bash
dotnet run --environment Staging
```

### Setting in Code (Rarely Used)

```csharp
var builder = WebApplication.CreateBuilder(new WebApplicationOptions
{
    EnvironmentName = Environments.Staging
});
```

> [!warning] Precedence Order
> The environment is resolved in this order (last one wins):
> 1. Host configuration defaults (Production)
> 2. `ASPNETCORE_ENVIRONMENT` environment variable
> 3. `launchSettings.json` (only when launched via `dotnet run` from the project directory)
> 4. Command-line arguments (`--environment`)
> 5. Explicit setting in code (`WebApplicationOptions.EnvironmentName`)

### Setting in IIS / Azure / Docker

For IIS, you set the environment variable in `web.config`:

```xml
<aspNetCore processPath="dotnet" arguments=".\OrderService.dll">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Production" />
  </environmentVariables>
</aspNetCore>
```

For Azure App Service, you set it in the Application Settings blade in the Azure portal.

For Docker:

```bash
docker run -e ASPNETCORE_ENVIRONMENT=Production my-order-service:latest
```

> [!summary] Section Summary
> - `ASPNETCORE_ENVIRONMENT` is the primary mechanism for setting the environment.
> - `launchSettings.json` is the standard approach during local development.
> - You can also set it via command-line arguments, code, IIS config, Azure settings, or Docker.
> - When unset, the environment defaults to Production for safety.
