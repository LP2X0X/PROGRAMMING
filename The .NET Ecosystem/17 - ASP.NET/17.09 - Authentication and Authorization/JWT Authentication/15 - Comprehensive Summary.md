---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


> [!tip] Complete Summary
> **JWT (JSON Web Token)** is a self-contained, stateless authentication mechanism ideal for APIs consumed by SPAs, mobile apps, and microservices. A JWT has three Base64Url-encoded parts -- Header (algorithm), Payload (claims), and Signature (integrity proof) -- separated by dots.
>
> **Setup** in ASP.NET Core involves adding the `Microsoft.AspNetCore.Authentication.JwtBearer` package, configuring `TokenValidationParameters` (issuer, audience, signing key, lifetime), and placing `UseAuthentication()` before `UseAuthorization()` in the middleware pipeline.
>
> **Token generation** creates claims, builds a `JwtSecurityToken` with signing credentials, and serializes it via `JwtSecurityTokenHandler`. **Login endpoints** validate credentials and return the token. **Refresh tokens** (opaque, database-stored) extend sessions without requiring re-authentication -- always implement rotation for security.
>
> **On the client**, HttpOnly cookies are the safest storage for browser apps; `localStorage` is convenient but vulnerable to XSS. **Claims** are accessed in controllers via `User.FindFirst()`, `User.Claims`, and `User.IsInRole()`.
>
> **Security essentials**: always use HTTPS, keep access tokens short-lived (15-30 min), use strong signing keys (256+ bits), never store secrets in the payload, implement key rotation, and understand that JWTs cannot be revoked before expiry -- pair them with revocable refresh tokens.
>
> The fundamental tradeoff: JWT trades **easy revocation** for **stateless scalability**. Choose JWT when you need cross-domain, multi-client, stateless authentication. Choose cookies when you need instant revocation and simpler browser security.
