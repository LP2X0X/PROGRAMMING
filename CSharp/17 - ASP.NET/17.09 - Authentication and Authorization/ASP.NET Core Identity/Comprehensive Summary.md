---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


> [!tip] Complete Summary
> **ASP.NET Core Identity** is a full-featured user management framework that provides:
>
> **Core Components:**
> - `UserManager<T>` -- CRUD operations on users, password management, claims, tokens
> - `SignInManager<T>` -- sign-in/sign-out, external login flows, 2FA coordination
> - `RoleManager<T>` -- role creation and management
> - `IdentityDbContext<T>` -- EF Core integration with 7 pre-defined tables
>
> **Out-of-the-Box Features:**
> - User registration and authentication
> - PBKDF2 password hashing (600,000 iterations in .NET 8+)
> - Email confirmation and password reset via secure tokens
> - TOTP-based two-factor authentication
> - Account lockout after configurable failed attempts
> - Role-based and claims-based authorization support
> - External login provider integration (OAuth 2.0 / OpenID Connect)
>
> **Three Registration Methods:**
> - `AddIdentity<TUser, TRole>` -- full stack with roles, no default UI
> - `AddDefaultIdentity<TUser>` -- includes default UI, no roles unless `.AddRoles<T>()`
> - `AddIdentityCore<TUser>` -- minimal, for APIs (UserManager only)
>
> **Key Setup Points:**
> - `UseAuthentication()` must precede `UseAuthorization()` in the middleware pipeline
> - Always call `base.OnModelCreating()` in your DbContext
> - Always check `IdentityResult.Succeeded` after mutations
> - Use `LocalRedirect` for return URLs to prevent open redirect attacks
> - Seed roles and admin users at application startup
>
> Identity is the right choice for most ASP.NET Core web applications. For simple API scenarios or non-EF stores, consider manual authentication with `AddIdentityCore` or custom middleware.
