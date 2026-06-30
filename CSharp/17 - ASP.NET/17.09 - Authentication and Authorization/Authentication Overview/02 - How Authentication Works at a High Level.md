---
tags:
  - csharp
  - asp-net-core
  - authentication
  - security
---


Understanding the complete flow from an incoming request to an authenticated user is essential. Here is the step-by-step process:

### The Complete Flow

```
Client Request
     |
     v
[1] Routing Middleware --> determines which endpoint will handle the request
     |
     v
[2] Authentication Middleware
     |  - Selects the default authentication handler
     |  - Handler inspects the request for credentials
     |    (reads cookie, extracts JWT from Authorization header, etc.)
     |  - If valid: creates ClaimsPrincipal, sets HttpContext.User
     |  - If invalid/missing: HttpContext.User remains anonymous
     |
     v
[3] Authorization Middleware
     |  - Checks [Authorize] requirements on the matched endpoint
     |  - If user is authenticated and authorized: continue
     |  - If user is not authenticated: CHALLENGE (401 / redirect to login)
     |  - If user is authenticated but not authorized: FORBID (403)
     |
     v
[4] Endpoint Execution (Controller Action / Minimal API handler)
```

### A Concrete Example: Cookie Authentication

```csharp
// 1. User submits login form
[HttpPost("login")]
public async Task<IActionResult> Login(LoginModel model)
{
    // 2. Validate credentials against your database
    var user = await _userService.ValidateCredentials(model.Email, model.Password);
    if (user is null)
        return View("Login", new { Error = "Invalid credentials" });

    // 3. Create claims describing the user
    var claims = new List<Claim>
    {
        new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
        new Claim(ClaimTypes.Name, user.DisplayName),
        new Claim(ClaimTypes.Email, user.Email),
        new Claim(ClaimTypes.Role, user.Role)
    };

    // 4. Create the identity and principal
    var identity = new ClaimsIdentity(claims, "CookieAuth");
    var principal = new ClaimsPrincipal(identity);

    // 5. Sign in -- this serializes the principal into an encrypted cookie
    await HttpContext.SignInAsync("Cookies", principal, new AuthenticationProperties
    {
        IsPersistent = model.RememberMe,
        ExpiresUtc = DateTimeOffset.UtcNow.AddHours(8)
    });

    return RedirectToAction("Index", "Home");
}

// 6. On subsequent requests, the cookie middleware automatically:
//    - Reads the cookie
//    - Decrypts it
//    - Rebuilds the ClaimsPrincipal
//    - Sets HttpContext.User
```

> [!ad-note]
> The `SignInAsync` method does not validate credentials. It takes an already-validated principal and persists it (usually as a cookie). Credential validation is **your** responsibility in the login action.

> [!summary] Section Summary
> The authentication flow runs on every request: the handler reads credentials, validates them, and builds a `ClaimsPrincipal`. For cookie auth, the initial sign-in serializes the principal into an encrypted cookie, and subsequent requests automatically deserialize it. The authorization middleware then uses the principal to enforce access rules.
