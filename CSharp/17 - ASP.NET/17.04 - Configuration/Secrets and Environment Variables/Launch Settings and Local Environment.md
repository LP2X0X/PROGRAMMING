---
tags:
  - csharp
  - asp-net-core
  - configuration
  - secrets
  - security
---


The `launchSettings.json` file (located in the `Properties` folder) lets you define environment variables for local development without polluting your system environment.

### Structure of launchSettings.json

```json
{
  "profiles": {
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "http://localhost:5000",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development",
        "Smtp__Password": "dev-password-from-launch-settings",
        "FeatureFlags__NewDashboard": "true"
      }
    },
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "https://localhost:5001;http://localhost:5000",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

> [!warning] Common Misconception
> `launchSettings.json` is **only** used by `dotnet run` and Visual Studio / Rider / VS Code launch configurations. It is **not** used when you deploy the application. Do not rely on it for staging or production configuration.

> [!tip] Pro Tip
> `launchSettings.json` is typically committed to git (it contains non-sensitive development URLs and settings). If you put secrets in the `environmentVariables` section, those secrets **will** be committed. Prefer User Secrets for development secrets instead.

### Setting Environment Variables in Different Contexts

**System-wide (Windows)**:

```powershell
# Permanent (requires admin for Machine scope)
[Environment]::SetEnvironmentVariable("Smtp__Password", "secret", "Machine")

# Current user only
[Environment]::SetEnvironmentVariable("Smtp__Password", "secret", "User")
```

**Docker**:

```dockerfile
# In Dockerfile
ENV ASPNETCORE_ENVIRONMENT=Production
ENV ConnectionStrings__DefaultConnection="Server=db;Database=MyApp;..."
```

```yaml
# In docker-compose.yml
services:
  web:
    image: myapp:latest
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - Smtp__Password=prod-secret
      - ConnectionStrings__DefaultConnection=Server=db;Database=MyApp;User=app;Password=secret
```

> [!example] Docker with .env file
> You can use a `.env` file with Docker Compose to avoid hardcoding secrets in `docker-compose.yml`:
> ```yaml
> # docker-compose.yml
> services:
>   web:
>     image: myapp:latest
>     env_file:
>       - .env
> ```
> ```bash
> # .env (add to .gitignore!)
> Smtp__Password=prod-secret
> ConnectionStrings__DefaultConnection=Server=db;Database=MyApp;User=app;Password=secret
> ```

**Azure App Service**:

In the Azure Portal, navigate to your App Service, then **Configuration** > **Application settings**. Each setting becomes an environment variable at runtime. You can also set them via the Azure CLI:

```bash
az webapp config appsettings set \
  --resource-group MyResourceGroup \
  --name MyAppService \
  --settings Smtp__Password="prod-secret"
```

> [!summary] Section Summary
> `launchSettings.json` sets environment variables for local `dotnet run` only -- never for deployment. System environment variables, Docker `ENV`/`environment`, and cloud platform configuration panels are used for deployed environments.
