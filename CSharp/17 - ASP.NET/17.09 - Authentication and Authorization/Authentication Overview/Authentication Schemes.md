---
tags:
  - csharp
  - asp-net-core
  - authentication
  - security
---


> [!info] Definition
> An **authentication scheme** is a named configuration that tells ASP.NET Core *how* to authenticate a request. Each scheme is associated with a specific handler that knows how to read and validate a particular credential type.

Think of schemes as different "strategies" for authentication. A cookie scheme knows how to read encrypted cookies. A JWT Bearer scheme knows how to validate JSON Web Tokens. An OAuth scheme knows how to redirect to an external provider.

### Registering Schemes

Schemes are registered during service configuration in `Program.cs`:

```csharp
builder.Services.AddAuthentication(defaultScheme: "Cookies")
    .AddCookie("Cookies", options =>
    {
        options.LoginPath = "/Account/Login";
        options.AccessDeniedPath = "/Account/AccessDenied";
        options.ExpireTimeSpan = TimeSpan.FromHours(2);
    })
    .AddJwtBearer("Bearer", options =>
    {
        options.Authority = "https://login.example.com";
        options.Audience = "my-api";
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true
        };
    });
```

The string passed to `AddCookie()` or `AddJwtBearer()` is the **scheme name**. You reference this name later when you need to specify which scheme to use.

### Common Built-In Schemes

| Scheme Type | NuGet Package | Common Use Case |
|---|---|---|
| Cookie | Built-in | Browser-based web apps |
| JWT Bearer | `Microsoft.AspNetCore.Authentication.JwtBearer` | APIs, SPAs, mobile apps |
| OAuth 2.0 | `Microsoft.AspNetCore.Authentication.OAuth` | External provider login |
| OpenID Connect | `Microsoft.AspNetCore.Authentication.OpenIdConnect` | Enterprise SSO, Azure AD |
| Google/Facebook/etc. | Individual packages | Social login |

> [!summary] Section Summary
> Authentication schemes are named strategies for reading and validating credentials. Each scheme maps to a handler. Multiple schemes can coexist in the same application, and you designate one as the default.
