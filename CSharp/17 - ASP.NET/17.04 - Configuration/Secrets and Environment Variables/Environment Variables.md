---
tags:
  - csharp
  - asp-net-core
  - configuration
  - secrets
  - security
---


**Environment variables** are the most universal mechanism for injecting configuration into an application. They work everywhere -- local machines, CI/CD pipelines, Docker containers, cloud platforms -- and ASP.NET Core reads them automatically.

### How ASP.NET Core Reads Environment Variables

`WebApplication.CreateBuilder(args)` adds environment variables to the configuration pipeline by default. Environment variables override values from JSON files because they are added later in the provider chain:

```
appsettings.json
  -> appsettings.{Environment}.json
    -> User Secrets (Development only)
      -> Environment variables        <-- higher priority
        -> Command-line arguments      <-- highest priority
```

> [!info] Definition
> The **configuration provider order** determines precedence. Providers added later override earlier ones. Environment variables sit near the top, meaning they override JSON file values for the same keys.

### Section Separators

Configuration keys in ASP.NET Core use `:` as the hierarchy separator (e.g., `Smtp:Password`). However, many operating systems do not allow `:` in environment variable names.

| Platform | Separator | Example |
|---|---|---|
| Windows | `__` (double underscore) or `:` | `Smtp__Password=secret` |
| Linux / macOS | `__` (double underscore) | `Smtp__Password=secret` |
| All platforms (safe choice) | `__` (double underscore) | `Smtp__Password=secret` |

```bash
# Windows (Command Prompt)
set Smtp__Password=my-secret

# Windows (PowerShell)
$env:Smtp__Password = "my-secret"

# Linux / macOS
export Smtp__Password=my-secret
```

> [!warning] Common Misconception
> The colon `:` separator works on Windows but **not** on Linux/macOS for environment variable names. Always use `__` (double underscore) for cross-platform compatibility. ASP.NET Core automatically converts `__` to `:` when reading configuration.

### The ASPNETCORE_ Prefix

ASP.NET Core recognizes a set of **framework-specific** environment variables with the `ASPNETCORE_` prefix. These control framework behavior, not application configuration:

| Variable | Purpose | Example |
|---|---|---|
| `ASPNETCORE_ENVIRONMENT` | Sets the hosting environment | `Development`, `Staging`, `Production` |
| `ASPNETCORE_URLS` | Sets the URLs the server listens on | `http://localhost:5000;https://localhost:5001` |
| `ASPNETCORE_HTTPS_PORT` | Redirects HTTP to this HTTPS port | `443` |
| `ASPNETCORE_CONTENTROOT` | Sets the content root path | `/app` |

```bash
# Run the app in Staging mode
export ASPNETCORE_ENVIRONMENT=Staging
dotnet run
```

> [!tip] Pro Tip
> There is also a `DOTNET_` prefix for .NET runtime-level variables (e.g., `DOTNET_ENVIRONMENT`). When both `ASPNETCORE_ENVIRONMENT` and `DOTNET_ENVIRONMENT` are set, `ASPNETCORE_ENVIRONMENT` takes precedence for web apps.

### Reading Environment Variables in Code

Environment variables map directly into the configuration system:

```csharp
var builder = WebApplication.CreateBuilder(args);

// These all work -- the configuration system merges all sources
var smtpPassword = builder.Configuration["Smtp:Password"];
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
var environment = builder.Environment.EnvironmentName;
```

You can also read environment variables directly (bypassing the configuration system), though this is rarely needed:

```csharp
var path = Environment.GetEnvironmentVariable("PATH");
```

> [!summary] Section Summary
> Environment variables are the universal secret-injection mechanism. Use `__` (double underscore) as the hierarchy separator for cross-platform safety. They override JSON configuration by default. Framework-specific variables use the `ASPNETCORE_` prefix.
