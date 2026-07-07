---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


| Aspect | ASP.NET Core Identity | Manual Authentication |
|---|---|---|
| **Scope** | Full user management framework | Custom authentication logic only |
| **Setup effort** | Low -- add packages, configure, migrate | High -- build everything yourself |
| **Password hashing** | Built-in PBKDF2 with secure defaults | You must implement or choose a library |
| **2FA** | Built-in TOTP support | You must integrate a 2FA library |
| **Email confirmation** | Built-in token generation | You must implement token logic |
| **External logins** | Built-in OAuth/OIDC integration | You must handle OAuth flows manually |
| **Roles and claims** | Full management system | You must build your own |
| **Database** | Requires EF Core (by default) | Use any data access layer |
| **Flexibility** | Opinionated but extensible | Complete control |
| **Boilerplate** | More tables, more abstractions | Only what you need |
| **Best for** | Most web applications | Simple APIs, non-EF stores, microservices |

> [!tip] When to Choose Manual Auth
> If you are building a simple API with JWT authentication and only need to verify credentials against a single table, manual auth with `[Authorize]` and a custom JWT middleware might be simpler than bringing in Identity. However, the moment you need password resets, email confirmation, 2FA, or role management, Identity will save you significant development time.

> [!summary] Section Summary
> Identity is a comprehensive, opinionated framework that handles nearly every auth scenario out of the box. Manual auth gives you more control and fewer dependencies but requires building everything yourself. Choose Identity for most web apps; choose manual auth for simple APIs or when you need to avoid the EF Core dependency.
