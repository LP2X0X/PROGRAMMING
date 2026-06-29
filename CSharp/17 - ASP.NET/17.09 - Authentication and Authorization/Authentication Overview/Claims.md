---
tags:
  - csharp
  - asp-net-core
  - authentication
  - security
---


> [!info] Definition
> A **Claim** is a key-value pair that states something about the user. Claims are facts asserted by the authentication authority -- such as "this user's name is john.doe" or "this user has the Admin role."

Claims are the building blocks of identity in ASP.NET Core. Rather than having a rigid `User` object with fixed properties, the claims model is flexible -- any piece of information about a user can be expressed as a claim.

### Standard Claim Types

The `System.Security.Claims.ClaimTypes` class defines well-known claim type URIs:

| Friendly Name | ClaimTypes Constant | URI |
|---|---|---|
| Name | `ClaimTypes.Name` | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` |
| Email | `ClaimTypes.Email` | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |
| Role | `ClaimTypes.Role` | `http://schemas.microsoft.com/ws/2008/06/identity/claims/role` |
| NameIdentifier | `ClaimTypes.NameIdentifier` | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier` |

### Creating Claims

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, "john.doe"),
    new Claim(ClaimTypes.Email, "john@example.com"),
    new Claim(ClaimTypes.Role, "Admin"),
    new Claim(ClaimTypes.Role, "Editor"),           // Multiple roles
    new Claim(ClaimTypes.NameIdentifier, "12345"),  // Unique user ID
    new Claim("Department", "Engineering"),          // Custom claim
    new Claim("EmployeeId", "EMP-0042"),             // Custom claim
    new Claim("HireDate", "2023-01-15")              // Custom claim
};
```

### Reading Claims

```csharp
// Find first matching claim
string? email = User.FindFirst(ClaimTypes.Email)?.Value;

// Find all matching claims (useful for roles)
IEnumerable<string> roles = User.FindAll(ClaimTypes.Role)
    .Select(c => c.Value);

// Check if a claim exists with a specific value
bool isAdmin = User.HasClaim(ClaimTypes.Role, "Admin");

// Check if a claim of a type exists at all
bool hasEmail = User.HasClaim(c => c.Type == ClaimTypes.Email);
```

> [!warning] Common Misconception
> Claims are **not** permissions. A claim states a fact about the user ("this user is in the Admin role"). Authorization policies then *interpret* those claims to make access decisions. The claim itself does not grant or deny anything.

> [!summary] Section Summary
> Claims are key-value pairs that describe the user. Standard claim types exist for common attributes like Name, Email, and Role, but you can define custom claims for any purpose. Claims are read from the `ClaimsPrincipal` using `FindFirst()`, `FindAll()`, and `HasClaim()`.
