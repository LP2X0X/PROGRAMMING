---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


The sign-in process involves creating the user's identity and calling `HttpContext.SignInAsync()` to issue the cookie.

### The SignInAsync Method

```csharp
await HttpContext.SignInAsync(
    scheme: CookieAuthenticationDefaults.AuthenticationScheme,
    principal: claimsPrincipal,
    properties: new AuthenticationProperties
    {
        IsPersistent = true,               // Persistent cookie (survives browser close)
        ExpiresUtc = DateTimeOffset.UtcNow.AddHours(2),
        AllowRefresh = true                // Allow sliding expiration
    }
);
```

### Complete Sign-In Process

```csharp
// 1. Build the claims
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.DisplayName),
    new Claim(ClaimTypes.Email, user.Email),
    new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
    new Claim(ClaimTypes.Role, "Admin"),
    new Claim("Department", "Engineering")  // Custom claim
};

// 2. Create an identity from those claims
var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);

// 3. Create a principal from the identity
var principal = new ClaimsPrincipal(identity);

// 4. Sign in -- this encrypts the principal into a cookie
await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    principal,
    new AuthenticationProperties
    {
        IsPersistent = rememberMe
    }
);
```

> [!ad-note]
> The second parameter of `ClaimsIdentity` constructor -- the `authenticationType` string -- is critical. If you pass `null` or omit it, the identity's `IsAuthenticated` property will return `false`, and the user will appear unauthenticated even though they have a valid cookie. Always pass the authentication scheme name or any non-null string.

> [!summary] Section Summary
> Sign-in requires building a claims list, creating a `ClaimsIdentity` with an authentication type, wrapping it in a `ClaimsPrincipal`, and calling `HttpContext.SignInAsync()`.
