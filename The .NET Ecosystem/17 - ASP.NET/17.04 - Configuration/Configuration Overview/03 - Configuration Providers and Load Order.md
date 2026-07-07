---
tags:
  - csharp
  - asp-net-core
  - configuration
  - fundamentals
---


**Configuration providers** are the pluggable components that read settings from a source and feed them into the configuration system. `WebApplication.CreateBuilder()` registers the following providers **in this default order**:

| Order | Provider | Source | Override Scope |
|---|---|---|---|
| 1 | `JsonConfigurationProvider` | `appsettings.json` | Base / shared settings |
| 2 | `JsonConfigurationProvider` | `appsettings.{Environment}.json` | Per-environment overrides |
| 3 | `UserSecretsConfigurationProvider` | `secrets.json` (local machine) | Development-only secrets |
| 4 | `EnvironmentVariablesConfigurationProvider` | OS environment variables | Deployment / container config |
| 5 | `CommandLineConfigurationProvider` | CLI arguments | One-off runtime overrides |

> [!warning] Override Rule
> The override rule is simple but critical: **the provider registered last wins**. If `appsettings.json` sets `MaxRetries` to `3` and an environment variable sets it to `5`, the value is `5` because environment variables are loaded after JSON files.

### Provider Details

### 1. appsettings.json

The base JSON file that ships with every project. Contains default values suitable for all environments.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  },
  "MaxRetries": 3
}
```

### 2. appsettings.{Environment}.json

An environment-specific file (e.g., `appsettings.Development.json`, `appsettings.Production.json`). The environment name comes from the `ASPNETCORE_ENVIRONMENT` variable.

```json
// appsettings.Development.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  },
  "MaxRetries": 10
}
```

> [!tip] File Is Optional
> Environment-specific JSON files are loaded with `optional: true`. If `appsettings.Staging.json` does not exist, the app starts normally using the base `appsettings.json` values.

### 3. User Secrets (Development Only)

**User secrets** store sensitive values (API keys, connection strings) outside the project directory so they are never committed to source control. They are only loaded when the environment is `Development`.

```bash
dotnet user-secrets init
dotnet user-secrets set "Smtp:Password" "my-secret-password"
```

Secrets are stored in a per-user location:
- Windows: `%APPDATA%\Microsoft\UserSecrets\<user_secrets_id>\secrets.json`
- Linux/macOS: `~/.microsoft/usersecrets/<user_secrets_id>/secrets.json`

> [!danger] Never Commit Secrets
> User secrets are a **development-only** convenience. In production, use environment variables, Azure Key Vault, AWS Secrets Manager, or another secure store. Never put real credentials in `appsettings.json`.

### 4. Environment Variables

Environment variables are the standard mechanism for injecting configuration in containers, CI/CD pipelines, and cloud platforms. Hierarchical keys use a **double underscore** (`__`) as the delimiter instead of a colon because colons are not valid in environment variable names on all platforms.

```bash
# Sets the key "Smtp:Host" in configuration
export Smtp__Host=smtp.example.com

# Sets the key "ConnectionStrings:Default"
export ConnectionStrings__Default="Server=prod-db;Database=App;..."
```

> [!tip] Prefix Filtering
> You can scope environment variables with a prefix to avoid conflicts:
> ```csharp
> builder.Configuration.AddEnvironmentVariables(prefix: "MYAPP_");
> ```
> Only variables starting with `MYAPP_` are loaded, and the prefix is stripped from the key.

### 5. Command-Line Arguments

The highest-priority default provider. Useful for one-off overrides when launching the app.

```bash
dotnet run --MaxRetries 20 --Smtp:Host=smtp.override.com
```

Supported formats:
- `--key value`
- `--key=value`
- `/key value`
- `/key=value`

> [!summary] Section Summary
> Five default providers are loaded in a fixed order: `appsettings.json`, `appsettings.{Environment}.json`, user secrets (Development only), environment variables, and command-line arguments. Later providers override earlier ones, so command-line arguments have the highest priority.
