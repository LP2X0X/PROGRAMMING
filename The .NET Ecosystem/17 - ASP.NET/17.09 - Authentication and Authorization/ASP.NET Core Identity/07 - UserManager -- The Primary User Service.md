---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


> [!info] Definition
> **`UserManager<TUser>`** is the primary service for performing CRUD operations on user accounts. It is injected via DI and provides methods for creating users, managing passwords, handling claims, generating tokens, and more.

## Common Operations

### Creating a User

```csharp
public class AccountService
{
    private readonly UserManager<ApplicationUser> _userManager;

    public AccountService(UserManager<ApplicationUser> userManager)
    {
        _userManager = userManager;
    }

    public async Task<IdentityResult> RegisterUserAsync(
        string email, string password, string fullName)
    {
        var user = new ApplicationUser
        {
            UserName = email,
            Email = email,
            FullName = fullName,
            HireDate = DateTime.UtcNow
        };

        // CreateAsync hashes the password and saves the user
        IdentityResult result = await _userManager.CreateAsync(user, password);

        if (result.Succeeded)
        {
            // Optionally assign a default role
            await _userManager.AddToRoleAsync(user, "Employee");
        }

        return result;
    }
}
```

> [!warning] Common Misconception
> `CreateAsync` returns an `IdentityResult`, not the user. You must check `result.Succeeded` before proceeding. If it fails (e.g., password too weak, duplicate email), the errors are in `result.Errors` -- a collection of `IdentityError` objects with `Code` and `Description` properties.

### Finding Users

```csharp
// By email
ApplicationUser? user = await _userManager.FindByEmailAsync("john@example.com");

// By ID
ApplicationUser? user = await _userManager.FindByIdAsync("some-guid-string");

// By username
ApplicationUser? user = await _userManager.FindByNameAsync("john@example.com");
```

### Password Operations

```csharp
// Verify a password without signing in
bool isValid = await _userManager.CheckPasswordAsync(user, "MyPassword123!");

// Change password (requires old password)
IdentityResult result = await _userManager.ChangePasswordAsync(
    user, "OldPassword", "NewPassword123!");

// Reset password (uses a token -- for "forgot password" flows)
string token = await _userManager.GeneratePasswordResetTokenAsync(user);
// ... send token to user via email ...
IdentityResult result = await _userManager.ResetPasswordAsync(
    user, token, "NewPassword123!");
```

### Claims Management

```csharp
// Add a claim to a user
await _userManager.AddClaimAsync(user, new Claim("Department", "Engineering"));

// Get all claims for a user
IList<Claim> claims = await _userManager.GetClaimsAsync(user);

// Remove a claim
await _userManager.RemoveClaimAsync(user, existingClaim);
```

### Email Confirmation

```csharp
// Generate confirmation token
string token = await _userManager.GenerateEmailConfirmationTokenAsync(user);
// ... send email with token ...

// Confirm the email when user clicks the link
IdentityResult result = await _userManager.ConfirmEmailAsync(user, token);

// Check if email is confirmed
bool confirmed = await _userManager.IsEmailConfirmedAsync(user);
```

## Key UserManager Methods Reference

| Method | Purpose |
|---|---|
| `CreateAsync(user, password)` | Create a user with a hashed password |
| `DeleteAsync(user)` | Delete a user |
| `UpdateAsync(user)` | Save changes to a user |
| `FindByEmailAsync(email)` | Find user by email |
| `FindByIdAsync(id)` | Find user by ID |
| `CheckPasswordAsync(user, password)` | Verify a password |
| `AddToRoleAsync(user, role)` | Assign a role |
| `RemoveFromRoleAsync(user, role)` | Remove a role |
| `GetRolesAsync(user)` | Get user's roles |
| `IsInRoleAsync(user, role)` | Check role membership |
| `AddClaimAsync(user, claim)` | Add a claim |
| `GetClaimsAsync(user)` | Get claims |
| `GenerateEmailConfirmationTokenAsync(user)` | Token for email confirmation |
| `ConfirmEmailAsync(user, token)` | Confirm email |
| `GeneratePasswordResetTokenAsync(user)` | Token for password reset |
| `ResetPasswordAsync(user, token, newPassword)` | Reset password with token |

> [!summary] Section Summary
> `UserManager<T>` is the central service for all user operations. It provides methods for CRUD, password management, role assignment, claims management, and token generation. Always check `IdentityResult.Succeeded` after mutating operations.
