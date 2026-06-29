---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


> [!info] Definition
> **`SignInManager<TUser>`** handles the sign-in and sign-out process. It works with the authentication middleware to issue and revoke authentication cookies.

## Password Sign-In

```csharp
public class AuthController : Controller
{
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly UserManager<ApplicationUser> _userManager;

    public AuthController(
        SignInManager<ApplicationUser> signInManager,
        UserManager<ApplicationUser> userManager)
    {
        _signInManager = signInManager;
        _userManager = userManager;
    }

    [HttpPost]
    public async Task<IActionResult> Login(LoginViewModel model)
    {
        if (!ModelState.IsValid)
            return View(model);

        var result = await _signInManager.PasswordSignInAsync(
            model.Email,
            model.Password,
            isPersistent: model.RememberMe,   // persistent cookie
            lockoutOnFailure: true             // increment failed count
        );

        if (result.Succeeded)
            return RedirectToAction("Index", "Home");

        if (result.IsLockedOut)
            return View("Lockout");

        if (result.RequiresTwoFactor)
            return RedirectToAction("LoginWith2fa");

        if (result.IsNotAllowed)
        {
            // Email not confirmed, etc.
            ModelState.AddModelError("", "Login not allowed. Confirm your email first.");
            return View(model);
        }

        ModelState.AddModelError("", "Invalid login attempt.");
        return View(model);
    }
}
```

## Sign-In Result Properties

| Property | Meaning |
|---|---|
| `Succeeded` | Login was successful |
| `IsLockedOut` | Account is currently locked out |
| `RequiresTwoFactor` | User has 2FA enabled, needs second factor |
| `IsNotAllowed` | Sign-in not allowed (e.g., unconfirmed email when `RequireConfirmedEmail = true`) |

## Signing Out

```csharp
[HttpPost]
public async Task<IActionResult> Logout()
{
    await _signInManager.SignOutAsync();
    return RedirectToAction("Index", "Home");
}
```

## Checking If a User Is Signed In

```csharp
// In a controller or Razor Page
if (_signInManager.IsSignedIn(User))
{
    // User is authenticated
}

// In a Razor view
@if (SignInManager.IsSignedIn(User))
{
    <a asp-action="Logout">Log Out</a>
}
```

## External Login Flow

```csharp
// Step 1: Challenge -- redirect to external provider
[HttpPost]
public IActionResult ExternalLogin(string provider)
{
    var redirectUrl = Url.Action("ExternalLoginCallback");
    var properties = _signInManager.ConfigureExternalAuthenticationProperties(
        provider, redirectUrl);
    return Challenge(properties, provider);
}

// Step 2: Callback -- handle the response
[HttpGet]
public async Task<IActionResult> ExternalLoginCallback()
{
    var info = await _signInManager.GetExternalLoginInfoAsync();
    if (info == null)
        return RedirectToAction("Login");

    // Try to sign in with the external login
    var result = await _signInManager.ExternalLoginSignInAsync(
        info.LoginProvider, info.ProviderKey, isPersistent: false);

    if (result.Succeeded)
        return RedirectToAction("Index", "Home");

    // If user doesn't exist yet, create an account
    var email = info.Principal.FindFirstValue(ClaimTypes.Email);
    // ... create user and link external login ...
}
```

> [!summary] Section Summary
> `SignInManager<T>` handles authentication flows -- password sign-in, sign-out, external logins, and 2FA. `PasswordSignInAsync` returns a rich result object indicating success, lockout, 2FA requirement, or denial. Always check all result properties to handle each scenario.
