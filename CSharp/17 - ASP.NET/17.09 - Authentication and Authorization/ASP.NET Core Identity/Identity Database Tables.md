---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


When you run EF Core migrations with Identity, the following tables are created:

| Table | Purpose | Key Columns |
|---|---|---|
| **AspNetUsers** | Stores user accounts | Id, UserName, Email, PasswordHash, SecurityStamp, plus your custom columns |
| **AspNetRoles** | Defines available roles | Id, Name, NormalizedName |
| **AspNetUserRoles** | Maps users to roles (many-to-many) | UserId, RoleId |
| **AspNetUserClaims** | Stores claims attached directly to users | Id, UserId, ClaimType, ClaimValue |
| **AspNetRoleClaims** | Stores claims attached to roles (inherited by role members) | Id, RoleId, ClaimType, ClaimValue |
| **AspNetUserLogins** | Associates external login providers with user accounts | LoginProvider, ProviderKey, UserId |
| **AspNetUserTokens** | Stores tokens (2FA recovery codes, authenticator keys) | UserId, LoginProvider, Name, Value |

## How the Tables Relate

```
AspNetUsers (1) ----< (many) AspNetUserRoles (many) >---- (1) AspNetRoles
     |                                                          |
     |---< AspNetUserClaims                                     |---< AspNetRoleClaims
     |---< AspNetUserLogins
     |---< AspNetUserTokens
```

> [!tip] Claims From Roles
> When a user belongs to a role, they automatically inherit that role's claims from `AspNetRoleClaims`. This is why role claims are powerful -- you can define a set of permissions on a role and every user in that role gets them without individual claim assignments.

> [!ad-note] On the SecurityStamp Column
> `SecurityStamp` in `AspNetUsers` is a critical security feature. It changes whenever a user's credentials change (password, email, 2FA settings). Existing authentication cookies are validated against this stamp, so changing your password immediately invalidates all your other sessions.

> [!summary] Section Summary
> Identity creates seven tables: AspNetUsers, AspNetRoles, AspNetUserRoles, AspNetUserClaims, AspNetRoleClaims, AspNetUserLogins, and AspNetUserTokens. Together, they store the complete user management data model including accounts, roles, claims, external logins, and tokens.
