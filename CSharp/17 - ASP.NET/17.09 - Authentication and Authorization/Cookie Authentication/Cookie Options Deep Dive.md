---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


The `CookieAuthenticationOptions` class provides extensive control over how cookies are created, transmitted, and validated.

### Options Reference Table

| Option | Type | Default | Description |
|---|---|---|---|
| `LoginPath` | `PathString` | `/Account/Login` | Redirect path for unauthenticated users |
| `LogoutPath` | `PathString` | `/Account/Logout` | Path for logout handling |
| `AccessDeniedPath` | `PathString` | `/Account/AccessDenied` | Redirect path for unauthorized users |
| `ExpireTimeSpan` | `TimeSpan` | 14 days | How long the authentication ticket is valid |
| `SlidingExpiration` | `bool` | `true` | Whether to renew expiration on active requests |
| `ReturnUrlParameter` | `string` | `"ReturnUrl"` | Query string parameter name for the return URL |
| `Cookie.Name` | `string` | `.AspNetCore.Cookies` | The name of the cookie |
| `Cookie.HttpOnly` | `bool` | `true` | If `true`, cookie is inaccessible to JavaScript |
| `Cookie.SecurePolicy` | `CookieSecurePolicy` | `SameAsRequest` | When to send the cookie over HTTPS only |
| `Cookie.SameSite` | `SameSiteMode` | `Lax` | CSRF protection mode |
| `Cookie.Domain` | `string` | (not set) | Domain the cookie is valid for |
| `Cookie.Path` | `string` | `/` | Path the cookie is valid for |
| `Cookie.MaxAge` | `TimeSpan?` | `null` | Browser-level max age of the cookie |

### Detailed Cookie Properties

```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        // Redirect paths
        options.LoginPath = "/Account/Login";
        options.LogoutPath = "/Account/Logout";
        options.AccessDeniedPath = "/Account/AccessDenied";
        options.ReturnUrlParameter = "returnUrl";

        // Ticket lifetime
        options.ExpireTimeSpan = TimeSpan.FromHours(2);
        options.SlidingExpiration = true;

        // Cookie properties
        options.Cookie.Name = "MyApp.Auth";
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.SameSite = SameSiteMode.Strict;
        options.Cookie.Domain = ".myapp.com";
        options.Cookie.Path = "/";
        options.Cookie.MaxAge = TimeSpan.FromHours(2);
    });
```

> [!warning] Common Misconception
> `ExpireTimeSpan` and `Cookie.MaxAge` are **not** the same thing. `ExpireTimeSpan` controls the authentication ticket validity inside the encrypted cookie. `Cookie.MaxAge` controls how long the browser keeps the cookie. If the browser cookie has not expired but the ticket inside it has, the user will still need to log in again. Best practice is to keep them aligned or rely on `ExpireTimeSpan` alone.

> [!tip] Production Best Practice
> In production, always set `Cookie.SecurePolicy = CookieSecurePolicy.Always` and `Cookie.SameSite = SameSiteMode.Strict` (or at minimum `Lax`). These settings prevent the cookie from being sent over insecure connections and provide protection against CSRF attacks.

> [!summary] Section Summary
> Cookie options control redirect paths, ticket lifetime, sliding expiration, and the physical cookie properties (name, security flags, SameSite mode). Configure these carefully for production security.
