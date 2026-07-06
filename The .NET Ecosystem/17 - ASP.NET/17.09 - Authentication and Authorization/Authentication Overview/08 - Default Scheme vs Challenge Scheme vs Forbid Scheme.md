---
tags:
  - csharp
  - asp-net-core
  - authentication
  - security
---


ASP.NET Core uses several distinct "scheme selectors" to determine which handler to invoke for different operations. Understanding these is critical for applications with multiple authentication schemes.

### The Four Scheme Types

| Scheme Type | Purpose | When It Fires |
|---|---|---|
| **DefaultScheme** | Fallback for all operations if no specific scheme is set | Any operation without an explicit scheme |
| **DefaultAuthenticateScheme** | Which handler reads credentials from the request | Every request (via authentication middleware) |
| **DefaultChallengeScheme** | What happens when an unauthenticated user hits a protected resource | Authorization middleware detects no authenticated user |
| **DefaultForbidScheme** | What happens when an authenticated user lacks permission | Authorization middleware detects insufficient permissions |

### Behavior of Each

**Authenticate** -- the handler that reads and validates credentials on every request:
```csharp
// The cookie handler reads the auth cookie and builds the principal
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = "Cookies";
});
```

**Challenge** -- invoked when an unauthenticated user accesses a protected endpoint. For cookies, this typically means redirecting to a login page. For JWT, this means returning a `401 Unauthorized` response:
```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultChallengeScheme = "Cookies"; // Redirects to login
    // vs.
    options.DefaultChallengeScheme = "Bearer";  // Returns 401
});
```

**Forbid** -- invoked when an authenticated user lacks permission. For cookies, this redirects to an "Access Denied" page. For JWT, this returns `403 Forbidden`:
```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultForbidScheme = "Cookies"; // Redirects to Access Denied page
});
```

### Simplified Configuration

When you pass a single string to `AddAuthentication()`, it sets **all** default schemes at once:

```csharp
// This sets DefaultScheme, which is used as fallback for
// DefaultAuthenticateScheme, DefaultChallengeScheme,
// DefaultSignInScheme, DefaultSignOutScheme, and DefaultForbidScheme
builder.Services.AddAuthentication("Cookies")
    .AddCookie("Cookies");
```

> [!tip]
> For most applications with a single authentication scheme, just pass the scheme name to `AddAuthentication()`. You only need to set individual scheme types when mixing multiple schemes (e.g., cookies for browsers and JWT for APIs).

> [!summary] Section Summary
> ASP.NET Core distinguishes between Authenticate (read credentials), Challenge (handle unauthenticated access), and Forbid (handle unauthorized access) operations. Each can be mapped to a different handler. The `DefaultScheme` acts as a fallback for all operations when specific defaults are not set.
