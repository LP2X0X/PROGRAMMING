---
tags: [csharp, asp-net-core, configuration, secrets, security]
date: 2026-06-18
aliases: [User Secrets, Secret Management, Environment Variables in ASP.NET]
status: complete
---

# Secrets and Environment Variables

> [!abstract] Overview
> ASP.NET Core provides a layered configuration system that separates sensitive data (connection strings, API keys, passwords) from application code. This note covers the full spectrum of secret management: **User Secrets** for local development, **environment variables** for deployment flexibility, **cloud secret stores** for production, and the workflows that tie them together. The goal is simple -- secrets should never exist in source control.

---

## Table of Contents

- [[#The Problem -- Secrets in Source Control]]
- [[#User Secrets (Development Only)]]
- [[#Environment Variables]]
- [[#Launch Settings and Local Environment]]
- [[#Cloud Secret Stores (Production)]]
- [[#.env Files]]
- [[#What NEVER to Do]]
- [[#Real-World Workflow]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]
- [[#Further Reading]]

---

## The Problem -- Secrets in Source Control

Every application needs sensitive configuration: database connection strings, SMTP passwords, API keys for third-party services, OAuth client secrets. The most natural place to put them is `appsettings.json` -- and that is exactly the wrong place.

```json
// appsettings.json -- DO NOT DO THIS
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=prod-db;Database=MyApp;User=sa;Password=SuperSecret123!"
  },
  "Smtp": {
    "Host": "smtp.example.com",
    "Port": 587,
    "Username": "noreply@example.com",
    "Password": "email-password-here"
  },
  "Stripe": {
    "SecretKey": "sk_live_xxxxxxxxxxxxxxxxxxxxxxxx"
  }
}
```

> [!danger] Critical Risk
> `appsettings.json` is tracked by git. Once a secret is committed, it lives in the repository history forever -- even if you delete it in a later commit. Automated bots scan public repositories for leaked credentials within minutes.

**The core principle**: configuration values that are not secret (logging levels, feature flags, non-sensitive URLs) belong in `appsettings.json`. Values that are secret belong in a **secret store** appropriate to the environment.

| Environment | Recommended Secret Store |
|---|---|
| Local development | User Secrets |
| CI/CD pipelines | Environment variables / pipeline secrets |
| Staging / Production | Azure Key Vault, AWS Secrets Manager, or env vars |
| Docker containers | Environment variables, Docker secrets |

> [!summary] Section Summary
> Secrets in `appsettings.json` get committed to git and exposed in repository history. Use purpose-built secret stores for each environment instead.

---

## User Secrets (Development Only)

**User Secrets** is a built-in ASP.NET Core feature that stores sensitive configuration in a JSON file outside the project directory -- in the developer's user profile folder. This means the secrets never appear in the project tree and cannot be accidentally committed.

### How It Works

User Secrets associates a project with a unique GUID (the **UserSecretsId**). At runtime, the configuration system reads from a JSON file keyed to that GUID, stored in the user's profile directory. The values from this file override values from `appsettings.json` and `appsettings.Development.json`.

> [!info] Definition
> **User Secrets** is a development-time configuration provider that stores key-value pairs in a JSON file located in the operating system's user profile directory, completely outside the project folder.

### Initializing User Secrets

Run the following command from the project directory (where the `.csproj` file lives):

```bash
dotnet user-secrets init
```

This adds a `UserSecretsId` element to your `.csproj` file:

```xml
<PropertyGroup>
  <TargetFramework>net8.0</TargetFramework>
  <UserSecretsId>a1b2c3d4-e5f6-7890-abcd-ef1234567890</UserSecretsId>
</PropertyGroup>
```

> [!tip] Pro Tip
> The GUID is just an identifier -- it has no cryptographic significance. Its sole purpose is to link your project to a specific secrets file on disk. You can share the same GUID across multiple projects if you want them to share secrets (though this is rarely useful).

### Setting Secrets

```bash
# Set a single secret (hierarchical keys use colon separator)
dotnet user-secrets set "Smtp:Password" "my-secret-password"

# Set a connection string
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=localhost;Database=MyApp;User=dev;Password=dev123"

# Set an API key
dotnet user-secrets set "Stripe:SecretKey" "sk_test_abc123"
```

### Listing Secrets

```bash
dotnet user-secrets list
```

Output:

```
Stripe:SecretKey = sk_test_abc123
Smtp:Password = my-secret-password
ConnectionStrings:DefaultConnection = Server=localhost;Database=MyApp;User=dev;Password=dev123
```

### Removing Secrets

```bash
# Remove a single secret
dotnet user-secrets remove "Smtp:Password"

# Remove all secrets for this project
dotnet user-secrets clear
```

### Where the File Lives

On Windows, the secrets file is stored at:

```
%APPDATA%\Microsoft\UserSecrets\<UserSecretsId>\secrets.json
```

For example:

```
C:\Users\YourName\AppData\Roaming\Microsoft\UserSecrets\a1b2c3d4-e5f6-7890-abcd-ef1234567890\secrets.json
```

On Linux and macOS:

```
~/.microsoft/usersecrets/<UserSecretsId>/secrets.json
```

The file itself is plain JSON with the same structure as `appsettings.json`:

```json
{
  "Smtp": {
    "Password": "my-secret-password"
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyApp;User=dev;Password=dev123"
  },
  "Stripe": {
    "SecretKey": "sk_test_abc123"
  }
}
```

> [!warning] Common Misconception
> User Secrets are **not encrypted**. They are stored as plain-text JSON. The protection comes from the file living outside the project directory (so it cannot be committed to git), not from encryption. Do not rely on User Secrets for production environments.

### Already Wired Up by Default

When you create a project with `WebApplication.CreateBuilder(args)`, User Secrets are automatically added to the configuration pipeline in the **Development** environment. You do not need any extra setup code:

```csharp
var builder = WebApplication.CreateBuilder(args);

// In Development, CreateBuilder() automatically calls:
// builder.Configuration.AddUserSecrets<Program>();
// You do NOT need to add this line yourself.

// Access secrets exactly like any other configuration
var smtpPassword = builder.Configuration["Smtp:Password"];
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
```

> [!tip] Pro Tip
> If you are using a non-web project (like a console app or worker service) and want User Secrets, you need to add the `Microsoft.Extensions.Configuration.UserSecrets` NuGet package and call `AddUserSecrets()` explicitly:
> ```csharp
> var config = new ConfigurationBuilder()
>     .AddJsonFile("appsettings.json")
>     .AddUserSecrets<Program>()
>     .Build();
> ```

### Using Secrets with the Options Pattern

User Secrets integrate seamlessly with strongly typed configuration via the [[Options Pattern]]:

```csharp
// SmtpOptions.cs
public class SmtpOptions
{
    public const string SectionName = "Smtp";

    public string Host { get; set; } = string.Empty;
    public int Port { get; set; }
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;  // Comes from User Secrets
}
```

```csharp
// Program.cs
builder.Services.Configure<SmtpOptions>(
    builder.Configuration.GetSection(SmtpOptions.SectionName));
```

```json
// appsettings.json -- non-sensitive values only
{
  "Smtp": {
    "Host": "smtp.example.com",
    "Port": 587,
    "Username": "noreply@example.com"
  }
}
```

```bash
# Secret stored via User Secrets
dotnet user-secrets set "Smtp:Password" "actual-password"
```

At runtime, the configuration system merges all sources. The `SmtpOptions` object receives `Host`, `Port`, and `Username` from `appsettings.json` and `Password` from User Secrets.

> [!summary] Section Summary
> User Secrets store sensitive configuration in a plain-text JSON file in the user profile directory, outside the project tree. Initialize with `dotnet user-secrets init`, manage with `set`, `list`, `remove`, and `clear`. Already wired up by default in `CreateBuilder()` for Development. Not encrypted -- protection comes from file location, not encryption.

---

## Environment Variables

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

---

## Launch Settings and Local Environment

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

---

## Cloud Secret Stores (Production)

For production environments, environment variables work but have limitations: they are visible in process listings, may appear in crash dumps, and managing hundreds of them across multiple services becomes unwieldy. **Cloud secret stores** provide encrypted storage, access control, versioning, and auditing.

### Azure Key Vault

**Azure Key Vault** is Microsoft's managed secret store. ASP.NET Core has first-class integration via the `Azure.Extensions.AspNetCore.Configuration.Secrets` NuGet package.

```bash
dotnet add package Azure.Extensions.AspNetCore.Configuration.Secrets
dotnet add package Azure.Identity
```

```csharp
var builder = WebApplication.CreateBuilder(args);

if (!builder.Environment.IsDevelopment())
{
    var keyVaultUri = new Uri(builder.Configuration["KeyVault:Uri"]!);
    builder.Configuration.AddAzureKeyVault(keyVaultUri, new DefaultAzureCredential());
}
```

Secrets stored in Key Vault with the name `Smtp--Password` (using `--` as separator) map to the configuration key `Smtp:Password`.

> [!tip] Pro Tip
> Use **Managed Identity** (`DefaultAzureCredential`) so your application authenticates to Key Vault without storing any credentials at all. The Azure platform handles identity transparently.

### AWS Secrets Manager

AWS provides **Secrets Manager** for the same purpose. Use the `AWSSDK.SecretsManager` NuGet package or community configuration providers:

```csharp
builder.Configuration.AddSecretsManager(configurator: options =>
{
    options.SecretFilter = entry => entry.Name.StartsWith("MyApp/");
    options.KeyGenerator = (_, name) => name.Replace("MyApp/", "").Replace("/", ":");
});
```

### HashiCorp Vault

For multi-cloud or on-premises scenarios, **HashiCorp Vault** is a popular choice. Community NuGet packages provide configuration provider integration.

| Feature | Azure Key Vault | AWS Secrets Manager | HashiCorp Vault |
|---|---|---|---|
| Managed service | Yes | Yes | Self-hosted or cloud |
| .NET integration | First-party NuGet | AWS SDK | Community packages |
| Secret versioning | Yes | Yes | Yes |
| Access auditing | Yes | Yes | Yes |
| Encryption | HSM-backed | AWS KMS | Configurable |

> [!summary] Section Summary
> Cloud secret stores (Azure Key Vault, AWS Secrets Manager, HashiCorp Vault) provide encrypted, audited, access-controlled secret management for production. They integrate into the ASP.NET Core configuration pipeline as additional providers.

---

## .env Files

**`.env` files** are not natively supported by ASP.NET Core, but they are extremely common in the broader ecosystem (Node.js, Python, Ruby) and are useful in ASP.NET Core projects that participate in polyglot environments.

### Third-Party Packages

The most common approach is to use a NuGet package like `dotenv.net`:

```bash
dotnet add package dotenv.net
```

```csharp
// Program.cs -- load .env file early
using dotenv.net;
DotEnv.Load();

var builder = WebApplication.CreateBuilder(args);
// Environment variables from .env are now available in the configuration system
```

### .env File Format

```bash
# .env file (place in project root)
ASPNETCORE_ENVIRONMENT=Development
Smtp__Password=my-dev-secret
ConnectionStrings__DefaultConnection=Server=localhost;Database=MyApp;User=dev;Password=dev123
Stripe__SecretKey=sk_test_abc123
```

> [!danger] Critical Warning
> The `.env` file **must** be added to `.gitignore`. It contains secrets and must never be committed.

```gitignore
# .gitignore
.env
.env.local
.env.*.local
```

A common pattern is to provide a `.env.example` file (committed to git) that documents the required variables without actual values:

```bash
# .env.example (committed to git -- no real secrets)
ASPNETCORE_ENVIRONMENT=Development
Smtp__Password=<your-smtp-password>
ConnectionStrings__DefaultConnection=<your-connection-string>
Stripe__SecretKey=<your-stripe-test-key>
```

> [!summary] Section Summary
> `.env` files are not built into ASP.NET Core but can be added via packages like `dotenv.net`. Always add `.env` to `.gitignore`. Provide a `.env.example` file with placeholder values as documentation.

---

## What NEVER to Do

> [!danger] Golden Rules of Secret Management
> 1. **Never commit secrets to git.** Once committed, a secret lives in the repository history forever.
> 2. **Never put real passwords in `appsettings.json`.** This file is committed and shared.
> 3. **Never put secrets in `launchSettings.json`.** This file is usually committed.
> 4. **Never log secrets.** Check that your logging configuration does not dump full configuration objects.
> 5. **Never embed secrets in code.** Hardcoded connection strings in C# files are just as dangerous as in JSON.
> 6. **Never share secrets via email, Slack, or Teams.** Use a secret manager or encrypted channel.

### If a Secret Was Already Committed

If you accidentally committed a secret, simply deleting it in a new commit is **not enough**. The secret remains in git history.

**Immediate steps**:

1. **Rotate the secret** -- generate a new password/key and invalidate the old one
2. **Remove from history** using `git filter-repo` or BFG Repo Cleaner (advanced)
3. **Force push** the cleaned history (requires team coordination)

```bash
# Using BFG Repo Cleaner to remove a file from all history
java -jar bfg.jar --delete-files secrets.json
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force
```

> [!warning] Warning
> Force-pushing rewrites history and affects all team members. Always coordinate with your team before doing this. And always rotate the compromised secret first -- removing it from history does not guarantee nobody already copied it.

### .gitignore Best Practices

Ensure these patterns are in your `.gitignore`:

```gitignore
# Secrets and local config
.env
.env.*
*.local
secrets.json
**/appsettings.Local.json

# User Secrets are already outside the project, but be safe
```

> [!summary] Section Summary
> Never commit secrets to git, embed them in code, or log them. If a secret is accidentally committed, rotate it immediately and then clean the git history. Maintain a thorough `.gitignore`.

---

## Real-World Workflow

Here is the complete workflow a team typically follows for managing secrets across environments.

### Development (Local Machine)

```bash
# 1. Clone the project
git clone https://github.com/team/my-app.git
cd my-app

# 2. Initialize User Secrets (already done if UserSecretsId is in .csproj)
dotnet user-secrets init

# 3. Set your local secrets
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=localhost;Database=MyApp;User=dev;Password=dev123"
dotnet user-secrets set "Smtp:Password" "local-dev-password"
dotnet user-secrets set "Stripe:SecretKey" "sk_test_abc123"

# 4. Run the app -- secrets are merged automatically
dotnet run
```

### CI/CD Pipeline (GitHub Actions Example)

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run tests
        env:
          ConnectionStrings__DefaultConnection: ${{ secrets.TEST_DB_CONNECTION }}
          Smtp__Password: ${{ secrets.SMTP_PASSWORD }}
        run: dotnet test

      - name: Deploy to Azure
        run: |
          az webapp config appsettings set \
            --resource-group MyRG \
            --name MyApp \
            --settings \
              ConnectionStrings__DefaultConnection="${{ secrets.PROD_DB_CONNECTION }}" \
              Smtp__Password="${{ secrets.PROD_SMTP_PASSWORD }}" \
              Stripe__SecretKey="${{ secrets.PROD_STRIPE_KEY }}"
```

> [!tip] Pro Tip
> In GitHub Actions, use **repository secrets** or **environment secrets** (Settings > Secrets and variables > Actions). They are encrypted at rest and masked in logs.

### Production (Azure Key Vault)

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Development: User Secrets (automatic)
// Production: Azure Key Vault
if (builder.Environment.IsProduction())
{
    var keyVaultUri = builder.Configuration["KeyVault:Uri"];
    if (!string.IsNullOrEmpty(keyVaultUri))
    {
        builder.Configuration.AddAzureKeyVault(
            new Uri(keyVaultUri),
            new DefaultAzureCredential());
    }
}

builder.Services.Configure<SmtpOptions>(
    builder.Configuration.GetSection("Smtp"));

var app = builder.Build();
```

### Summary Diagram

```
                     Configuration Sources (lowest to highest priority)
                     ================================================

  appsettings.json          Non-sensitive defaults (committed to git)
        |
  appsettings.{Env}.json   Environment-specific overrides (committed)
        |
  User Secrets              Development secrets (local machine only)
        |
  Environment Variables     Deployment secrets (CI/CD, Docker, cloud)
        |
  Azure Key Vault           Production secrets (encrypted, audited)
        |
  Command-line args         Highest priority override
```

> [!example] Complete Program.cs with All Layers
> ```csharp
> using Azure.Identity;
>
> var builder = WebApplication.CreateBuilder(args);
>
> // Layer 1-3: appsettings.json, appsettings.{env}.json, and User Secrets
> // are already added by CreateBuilder()
>
> // Layer 4: Environment variables are already added by CreateBuilder()
>
> // Layer 5: Azure Key Vault for production
> if (builder.Environment.IsProduction())
> {
>     var kvUri = builder.Configuration["KeyVault:Uri"];
>     if (!string.IsNullOrEmpty(kvUri))
>     {
>         builder.Configuration.AddAzureKeyVault(
>             new Uri(kvUri), new DefaultAzureCredential());
>     }
> }
>
> // Bind options -- they receive merged values from all layers
> builder.Services.Configure<SmtpOptions>(
>     builder.Configuration.GetSection("Smtp"));
> builder.Services.Configure<StripeOptions>(
>     builder.Configuration.GetSection("Stripe"));
>
> var app = builder.Build();
> app.Run();
> ```

> [!summary] Section Summary
> The real-world workflow uses User Secrets for local development, CI/CD pipeline secrets (like GitHub Actions secrets) for testing and deployment, and Azure Key Vault (or equivalent) for production. `CreateBuilder()` wires up most of the stack automatically.

---

## Comprehensive Summary

> [!tip] Complete Summary
> **The problem**: Secrets in `appsettings.json` get committed to git and exposed in repository history.
>
> **User Secrets** solve the development problem:
> - `dotnet user-secrets init` adds a GUID to `.csproj`
> - `dotnet user-secrets set "Key" "Value"` stores secrets in `%APPDATA%\Microsoft\UserSecrets\{id}\secrets.json`
> - Automatically loaded by `CreateBuilder()` in Development
> - Not encrypted -- protected by file location, not encryption
>
> **Environment variables** bridge development and production:
> - Use `__` (double underscore) as the hierarchy separator for cross-platform safety
> - Set via `launchSettings.json` (local only), system environment, Docker, or cloud platforms
> - `ASPNETCORE_` prefix for framework-specific variables
> - Override JSON configuration by default
>
> **Cloud secret stores** secure production:
> - Azure Key Vault, AWS Secrets Manager, HashiCorp Vault
> - Provide encryption, access control, auditing, and versioning
> - Integrate as configuration providers in the ASP.NET Core pipeline
>
> **`.env` files** are not built-in but available via third-party packages like `dotenv.net`. Always `.gitignore` them.
>
> **The golden rule**: secrets never touch source control. User Secrets for dev, environment variables or cloud vaults for prod.

---

## Related Topics

- [[Configuration Overview]] -- The full ASP.NET Core configuration system and provider chain
- [[Options Pattern]] -- Strongly typed configuration with `IOptions<T>`, `IOptionsSnapshot<T>`, and `IOptionsMonitor<T>`
- [[Strongly Typed Configuration]] -- Binding configuration sections to POCO classes

---

## Further Reading

- [Microsoft Docs: Safe storage of app secrets in development](https://learn.microsoft.com/en-us/aspnet/core/security/app-secrets)
- [Microsoft Docs: Configuration in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration)
- [Microsoft Docs: Azure Key Vault configuration provider](https://learn.microsoft.com/en-us/aspnet/core/security/key-vault-configuration)
- [Microsoft Docs: Use multiple environments](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/environments)
- [GitHub: dotenv.net](https://github.com/bolorundurowb/dotenv.net)
- [OWASP: Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
