---
tags:
  - csharp
  - asp-net-core
  - configuration
  - secrets
  - security
---


> [!abstract] Overview
> ASP.NET Core provides a layered configuration system that separates sensitive data (connection strings, API keys, passwords) from application code. This note covers the full spectrum of secret management: **User Secrets** for local development, **environment variables** for deployment flexibility, **cloud secret stores** for production, and the workflows that tie them together. The goal is simple -- secrets should never exist in source control.

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
