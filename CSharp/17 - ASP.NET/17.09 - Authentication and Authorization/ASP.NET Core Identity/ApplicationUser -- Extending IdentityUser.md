---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


## What IdentityUser Already Provides

The base `IdentityUser` class (from `Microsoft.AspNetCore.Identity`) comes with these properties:

| Property | Type | Description |
|---|---|---|
| `Id` | `string` | Unique identifier (GUID by default) |
| `UserName` | `string` | The username |
| `NormalizedUserName` | `string` | Uppercase version for case-insensitive lookups |
| `Email` | `string` | Email address |
| `NormalizedEmail` | `string` | Uppercase email for lookups |
| `EmailConfirmed` | `bool` | Whether email is verified |
| `PasswordHash` | `string` | The hashed password |
| `SecurityStamp` | `string` | Changes when credentials change (invalidates old tokens) |
| `ConcurrencyStamp` | `string` | Optimistic concurrency token |
| `PhoneNumber` | `string` | Phone number |
| `PhoneNumberConfirmed` | `bool` | Whether phone is verified |
| `TwoFactorEnabled` | `bool` | Whether 2FA is enabled |
| `LockoutEnd` | `DateTimeOffset?` | When the lockout expires |
| `LockoutEnabled` | `bool` | Whether lockout is allowed |
| `AccessFailedCount` | `int` | Count of failed login attempts |

## Creating a Custom User Class

You almost always want to extend `IdentityUser` with application-specific properties:

```csharp
public class ApplicationUser : IdentityUser
{
    [Required]
    [MaxLength(100)]
    public string FullName { get; set; } = string.Empty;

    [MaxLength(50)]
    public string Department { get; set; } = string.Empty;

    public DateTime HireDate { get; set; }

    public string? ProfilePictureUrl { get; set; }

    public bool IsActive { get; set; } = true;
}
```

> [!tip] Use ApplicationUser Everywhere
> Once you create `ApplicationUser`, use it consistently across your entire application -- in `AddIdentity<ApplicationUser, ...>`, in `UserManager<ApplicationUser>`, in `SignInManager<ApplicationUser>`, and in your `IdentityDbContext<ApplicationUser>`. Mixing up the generic type parameter is a common source of DI resolution errors.

> [!warning] Common Misconception
> The `Id` property on `IdentityUser` is a `string` by default, not an `int` or `Guid`. It stores a GUID as a string. If you want a different key type, you can inherit from `IdentityUser<TKey>` instead, e.g., `IdentityUser<int>` or `IdentityUser<Guid>`. But be aware this changes the generic parameters across the entire Identity stack.

> [!summary] Section Summary
> `IdentityUser` provides ~15 built-in properties including Id, UserName, Email, PasswordHash, and security-related fields. Extend it via `ApplicationUser : IdentityUser` to add domain-specific properties like Department and HireDate. Use `ApplicationUser` consistently in all Identity generics.
