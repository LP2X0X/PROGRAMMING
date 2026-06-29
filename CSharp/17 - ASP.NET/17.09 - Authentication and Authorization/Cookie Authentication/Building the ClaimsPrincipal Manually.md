---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


Understanding how to construct a `ClaimsPrincipal` from scratch is fundamental to cookie authentication. Here is a detailed breakdown of each component.

### Claims

A **claim** is a name-value pair representing a fact about the user. ASP.NET Core defines standard claim types in the `ClaimTypes` class.

```csharp
var claims = new List<Claim>
{
    // Standard claims
    new Claim(ClaimTypes.NameIdentifier, "12345"),
    new Claim(ClaimTypes.Name, "john.doe"),
    new Claim(ClaimTypes.Email, "john@example.com"),
    new Claim(ClaimTypes.GivenName, "John"),
    new Claim(ClaimTypes.Surname, "Doe"),
    new Claim(ClaimTypes.Role, "Admin"),
    new Claim(ClaimTypes.Role, "Manager"),  // Multiple roles are fine

    // Custom claims
    new Claim("Department", "Engineering"),
    new Claim("EmployeeId", "EMP-42"),
    new Claim("Tier", "Premium")
};
```

### ClaimsIdentity

A `ClaimsIdentity` bundles claims together with an authentication type. Think of it like an ID card -- it contains information and states what authority issued it.

```csharp
var identity = new ClaimsIdentity(
    claims: claims,
    authenticationType: CookieAuthenticationDefaults.AuthenticationScheme
);

// You can also specify which claim types map to Name and Role
var identity = new ClaimsIdentity(
    claims: claims,
    authenticationType: CookieAuthenticationDefaults.AuthenticationScheme,
    nameType: ClaimTypes.Name,
    roleType: ClaimTypes.Role
);
```

> [!info] Definition
> The `authenticationType` parameter determines the value of `ClaimsIdentity.IsAuthenticated`. If it is `null` or empty, `IsAuthenticated` returns `false`. If it is any non-null, non-empty string, `IsAuthenticated` returns `true`.

### ClaimsPrincipal

A `ClaimsPrincipal` represents the user and can hold multiple identities (for scenarios where the user authenticates through more than one scheme).

```csharp
var principal = new ClaimsPrincipal(identity);

// After sign-in, access claims anywhere via HttpContext:
var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
var userName = User.Identity?.Name;
var isAdmin = User.IsInRole("Admin");
var email = User.FindFirst(ClaimTypes.Email)?.Value;
```

> [!summary] Section Summary
> Build claims as a list of `Claim` objects, wrap them in a `ClaimsIdentity` with an authentication type, then wrap that in a `ClaimsPrincipal`. The principal represents the authenticated user throughout the request.
