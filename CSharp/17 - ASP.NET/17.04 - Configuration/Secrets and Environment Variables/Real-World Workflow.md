---
tags:
  - csharp
  - asp-net-core
  - configuration
  - secrets
  - security
---


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
