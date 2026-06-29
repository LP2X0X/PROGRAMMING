---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


ASP.NET Core can generate the complete set of Identity Razor Pages for you:

```bash
# Install the code generator tool
dotnet tool install -g dotnet-aspnet-codegenerator

# Scaffold all Identity pages
dotnet aspnet-codegenerator identity --dbContext ApplicationDbContext

# Scaffold specific pages only
dotnet aspnet-codegenerator identity --dbContext ApplicationDbContext \
    --files "Account.Login;Account.Register;Account.Logout;Account.ForgotPassword"
```

## What Gets Generated

Scaffolding creates a folder structure under `Areas/Identity/Pages/Account/`:

| Page | Purpose |
|---|---|
| `Register.cshtml` | User registration form |
| `Login.cshtml` | Login form |
| `Logout.cshtml` | Logout confirmation |
| `ForgotPassword.cshtml` | Initiate password reset |
| `ResetPassword.cshtml` | Enter new password with token |
| `ConfirmEmail.cshtml` | Email confirmation landing page |
| `Manage/Index.cshtml` | User profile management |
| `Manage/ChangePassword.cshtml` | Change password |
| `Manage/TwoFactorAuthentication.cshtml` | 2FA setup |
| `Manage/EnableAuthenticator.cshtml` | QR code for TOTP setup |

## When to Scaffold vs Write From Scratch

> [!example] Scaffold When:
> - You want a quick prototype with all auth flows working
> - Your UI requirements are close to the default -- just need styling changes
> - You want to override only a few specific pages (scaffold those pages only)

> [!example] Write From Scratch When:
> - You are building an API (no Razor Pages needed)
> - Your UI framework is React, Angular, or Blazor
> - You need significantly different UX flows (e.g., multi-step registration)
> - You want full control over every aspect of the auth experience

> [!summary] Section Summary
> `dotnet aspnet-codegenerator identity` scaffolds Razor Pages for all authentication flows. You can scaffold all pages or specific ones. Scaffolding is best for server-rendered apps with standard auth flows. For APIs or SPA frontends, write your own endpoints instead.
