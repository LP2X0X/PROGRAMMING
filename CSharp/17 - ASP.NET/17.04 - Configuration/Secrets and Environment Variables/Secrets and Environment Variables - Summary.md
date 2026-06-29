---
tags:
  - csharp
  - asp-net-core
  - configuration
  - secrets
  - security
---


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
