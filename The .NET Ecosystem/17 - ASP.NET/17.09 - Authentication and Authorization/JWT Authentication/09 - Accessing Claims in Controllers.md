---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


Once JWT authentication is configured, ASP.NET Core automatically populates `HttpContext.User` with the claims from the validated token.

## Getting a Specific Claim

```csharp
[Authorize]
[HttpGet("profile")]
public IActionResult GetProfile()
{
    // Get the user ID from the "sub" claim
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value
              ?? User.FindFirst(JwtRegisteredClaimNames.Sub)?.Value;

    // Get the email claim
    var email = User.FindFirst(ClaimTypes.Email)?.Value
             ?? User.FindFirst(JwtRegisteredClaimNames.Email)?.Value;

    // Get the role
    var role = User.FindFirst(ClaimTypes.Role)?.Value;

    return Ok(new { userId, email, role });
}
```

## Checking Roles

```csharp
[Authorize]
[HttpDelete("users/{id}")]
public IActionResult DeleteUser(int id)
{
    if (!User.IsInRole("Admin"))
    {
        return Forbid();
    }

    // ... delete logic
    return NoContent();
}
```

## Listing All Claims

```csharp
[Authorize]
[HttpGet("claims")]
public IActionResult GetAllClaims()
{
    var claims = User.Claims.Select(c => new
    {
        Type = c.Type,
        Value = c.Value
    });

    return Ok(claims);
}
```

> [!warning] Common Misconception
> ASP.NET Core maps JWT claim names to its own `ClaimTypes` constants. For example, `sub` becomes `ClaimTypes.NameIdentifier` and `email` becomes `ClaimTypes.Email`. If `User.FindFirst(ClaimTypes.NameIdentifier)` returns null, try the raw JWT claim name. You can disable this mapping:
> ```csharp
> JwtSecurityTokenHandler.DefaultInboundClaimTypeMap.Clear();
> ```

> [!example]
> **Using the `[Authorize]` attribute with roles:**
> ```csharp
> [Authorize(Roles = "Admin")]           // Only Admin
> [Authorize(Roles = "Admin,Manager")]   // Admin OR Manager
> ```
> Note: comma-separated roles in a single `[Authorize]` attribute are **OR** logic. For **AND** logic, use multiple attributes:
> ```csharp
> [Authorize(Roles = "Admin")]
> [Authorize(Roles = "Manager")]   // Must be BOTH Admin AND Manager
> ```

> [!summary] Section Summary
> Access JWT claims via `User.FindFirst()`, `User.Claims`, and `User.IsInRole()`. Be aware that ASP.NET Core remaps JWT claim names to `ClaimTypes` constants by default. Use `[Authorize(Roles = "...")]` for declarative role checks.
