---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


JWT defines a set of **registered claims** (standardized names) and allows **custom claims** for application-specific data.

## Registered (Standard) Claims

| Claim | Full Name | Description | Example |
|---|---|---|---|
| `sub` | Subject | The principal (usually user ID) | `"12345"` |
| `iss` | Issuer | Who issued the token | `"https://myapi.com"` |
| `aud` | Audience | Who the token is intended for | `"https://myapi.com"` |
| `exp` | Expiration Time | Unix timestamp after which the token is invalid | `1718740000` |
| `iat` | Issued At | Unix timestamp when the token was created | `1718736400` |
| `nbf` | Not Before | Unix timestamp before which the token is not valid | `1718736400` |
| `jti` | JWT ID | Unique identifier for the token (prevents replay) | `"a1b2c3d4-..."` |

## Common Custom Claims

| Claim | Description | Example |
|---|---|---|
| `email` | User's email address | `"john@example.com"` |
| `name` | User's display name | `"John Doe"` |
| `role` | User's role for authorization | `"Admin"` |
| `permissions` | Granular permission list | `["read", "write"]` |

> [!tip]
> In ASP.NET Core, use `JwtRegisteredClaimNames` for standard claims and `ClaimTypes` for framework-specific claims like `ClaimTypes.Role`. The `role` claim is special because ASP.NET Core maps it to enable `User.IsInRole()` checks.

> [!summary] Section Summary
> JWT has seven registered claims (`sub`, `iss`, `aud`, `exp`, `iat`, `nbf`, `jti`) and supports arbitrary custom claims. Use registered claims for interoperability and custom claims for application-specific data.
