---
tags:
  - csharp
  - asp-net-core
  - authentication
  - security
---


> [!info] Definition
> A **ClaimsPrincipal** represents the authenticated user. It is the top-level object accessible via `HttpContext.User` and contains one or more `ClaimsIdentity` objects.

The `ClaimsPrincipal` is the standard .NET security principal used across all of ASP.NET Core. Every request has one -- even anonymous requests (in which case the principal has no authenticated identity).

### Structure

```
ClaimsPrincipal
  |
  +-- ClaimsIdentity (e.g., from Cookie scheme)
  |     |-- Claim: Name = "john.doe"
  |     |-- Claim: Email = "john@example.com"
  |     |-- Claim: Role = "Admin"
  |
  +-- ClaimsIdentity (e.g., from external provider)
        |-- Claim: Name = "john.doe"
        |-- Claim: ProfilePicture = "https://..."
```

### Accessing the Principal

```csharp
// In a controller
public IActionResult Profile()
{
    var user = HttpContext.User;            // ClaimsPrincipal
    var name = user.Identity?.Name;        // Primary identity name
    var isAuth = user.Identity?.IsAuthenticated ?? false;

    var email = user.FindFirst(ClaimTypes.Email)?.Value;
    var roles = user.FindAll(ClaimTypes.Role).Select(c => c.Value);

    return View(new ProfileViewModel(name, email, roles));
}
```

> [!tip]
> `User.Identity.IsAuthenticated` returns `true` only if the identity has an `AuthenticationType` set. An identity created without specifying `AuthenticationType` is considered anonymous.

> [!summary] Section Summary
> `ClaimsPrincipal` is the root object representing the authenticated user. It holds one or more `ClaimsIdentity` objects, each containing claims. Access it via `HttpContext.User` in controllers and middleware.
