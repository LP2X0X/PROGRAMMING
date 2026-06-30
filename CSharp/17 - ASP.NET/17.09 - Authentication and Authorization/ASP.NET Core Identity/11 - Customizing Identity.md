---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


## Password Options (Detailed)

```csharp
builder.Services.Configure<IdentityOptions>(options =>
{
    // Password complexity
    options.Password.RequireDigit = true;             // Require at least one 0-9
    options.Password.RequiredLength = 8;              // Minimum length
    options.Password.RequireNonAlphanumeric = true;   // Require !@#$%^&* etc.
    options.Password.RequireUppercase = true;          // Require A-Z
    options.Password.RequireLowercase = true;          // Require a-z
    options.Password.RequiredUniqueChars = 4;          // Minimum distinct characters
});
```

## Lockout Options

```csharp
options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
options.Lockout.MaxFailedAccessAttempts = 5;
options.Lockout.AllowedForNewUsers = true;  // Apply lockout to new accounts too
```

## Cookie Settings

```csharp
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.Name = "MyApp.Auth";
    options.Cookie.HttpOnly = true;                    // Not accessible via JavaScript
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;  // HTTPS only
    options.Cookie.SameSite = SameSiteMode.Strict;
    options.ExpireTimeSpan = TimeSpan.FromHours(8);    // Cookie lifetime
    options.SlidingExpiration = true;                   // Refresh on activity
    options.LoginPath = "/Account/Login";               // Redirect path
    options.LogoutPath = "/Account/Logout";
    options.AccessDeniedPath = "/Account/AccessDenied";
});
```

## Token Lifespan

```csharp
builder.Services.Configure<DataProtectionTokenProviderOptions>(options =>
{
    options.TokenLifespan = TimeSpan.FromHours(3);  // Email/password reset tokens
});
```

> [!tip] Cookie HttpOnly and Secure
> Always set `HttpOnly = true` (prevents JavaScript access, mitigating XSS token theft) and `SecurePolicy = Always` (ensures cookies are only sent over HTTPS). These are security best practices, not optional settings.

> [!summary] Section Summary
> Identity is highly configurable. You can customize password complexity, lockout behavior, cookie properties (name, lifetime, security flags, paths), and token lifespans. Use `Configure<IdentityOptions>` for Identity settings and `ConfigureApplicationCookie` for cookie settings.
