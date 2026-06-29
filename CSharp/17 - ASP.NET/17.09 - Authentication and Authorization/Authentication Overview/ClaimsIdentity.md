---
tags:
  - csharp
  - asp-net-core
  - authentication
  - security
---


> [!info] Definition
> A **ClaimsIdentity** represents a single source of identity information. It contains a collection of claims and an `AuthenticationType` that indicates how the user was authenticated. Think of it as a single ID card.

A `ClaimsPrincipal` can hold multiple `ClaimsIdentity` objects -- just as a person might carry a driver's license, a passport, and a work badge. Each identity comes from a different authentication source and may contain different claims.

### Creating a ClaimsIdentity

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, "john.doe"),
    new Claim(ClaimTypes.Email, "john@example.com"),
    new Claim(ClaimTypes.Role, "Admin")
};

// The second parameter is the AuthenticationType
var identity = new ClaimsIdentity(claims, "CookieAuth");
```

> [!warning] Common Misconception
> If you create a `ClaimsIdentity` without specifying an `AuthenticationType`, the identity's `IsAuthenticated` property will return `false`. This is a common source of bugs -- the user has claims but appears unauthenticated:
> ```csharp
> // BUG: IsAuthenticated will be false!
> var identity = new ClaimsIdentity(claims);
> Console.WriteLine(identity.IsAuthenticated); // false
>
> // CORRECT: Specify AuthenticationType
> var identity = new ClaimsIdentity(claims, "CookieAuth");
> Console.WriteLine(identity.IsAuthenticated); // true
> ```

### Building a Principal from Identities

```csharp
// Identity from local login
var localClaims = new List<Claim>
{
    new Claim(ClaimTypes.Name, "john.doe"),
    new Claim(ClaimTypes.Role, "Admin")
};
var localIdentity = new ClaimsIdentity(localClaims, "LocalAuth");

// Identity from external provider (e.g., Google)
var externalClaims = new List<Claim>
{
    new Claim("picture", "https://lh3.google.com/..."),
    new Claim(ClaimTypes.Email, "john@gmail.com")
};
var externalIdentity = new ClaimsIdentity(externalClaims, "Google");

// Combine into a single principal
var principal = new ClaimsPrincipal(new[] { localIdentity, externalIdentity });
```

### Name and Role Claim Types

By default, `ClaimsIdentity` looks for `ClaimTypes.Name` when you access the `Name` property, and `ClaimTypes.Role` when checking roles. You can customize this:

```csharp
// Use custom claim types for name and role
var identity = new ClaimsIdentity(
    claims: claims,
    authenticationType: "Custom",
    nameType: "preferred_username",   // Instead of ClaimTypes.Name
    roleType: "groups"                // Instead of ClaimTypes.Role
);
```

> [!tip]
> JWT tokens from providers like Azure AD or Auth0 often use non-standard claim names (e.g., `preferred_username` instead of `ClaimTypes.Name`). Use the `nameType` and `roleType` parameters to map them correctly without manually transforming claims.

> [!summary] Section Summary
> A `ClaimsIdentity` groups claims from a single authentication source. It must have an `AuthenticationType` set to be considered authenticated. A `ClaimsPrincipal` can hold multiple identities, and the name/role claim type mappings are customizable per identity.
