---
tags:
  - csharp
  - asp-net-core
  - authentication
  - security
---


> [!info] Definition
> The `[AllowAnonymous]` attribute overrides `[Authorize]` to permit unauthenticated access to a specific endpoint. It is typically used on login pages, registration forms, and public API endpoints within an otherwise protected controller.

### Usage

```csharp
[Authorize]  // All actions require authentication by default
public class AccountController : Controller
{
    // Requires authentication (inherits from controller)
    public IActionResult Profile() => View();

    // Overrides [Authorize] -- accessible without authentication
    [AllowAnonymous]
    public IActionResult Login() => View();

    // Also accessible without authentication
    [AllowAnonymous]
    [HttpPost]
    public async Task<IActionResult> Login(LoginModel model)
    {
        // Validate credentials and sign in
        return RedirectToAction("Profile");
    }

    // Accessible without authentication
    [AllowAnonymous]
    public IActionResult Register() => View();
}
```

### With Global Fallback Policy

When you set a global `FallbackPolicy`, `[AllowAnonymous]` becomes essential for public endpoints:

```csharp
// Program.cs -- require auth everywhere by default
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});
```

```csharp
// Public endpoints must explicitly opt out
public class HomeController : Controller
{
    [AllowAnonymous]
    public IActionResult Index() => View();  // Landing page

    [AllowAnonymous]
    public IActionResult Privacy() => View();

    // No attribute needed -- FallbackPolicy applies
    public IActionResult Dashboard() => View(); // Requires auth
}
```

### Minimal API Equivalent

```csharp
app.MapGet("/public", () => "Hello, World!")
    .AllowAnonymous();

app.MapGet("/protected", () => "Secret data")
    .RequireAuthorization();
```

> [!warning] Common Misconception
> `[AllowAnonymous]` does **not** prevent authentication from running. If a user sends a valid cookie or token, the authentication middleware will still populate `HttpContext.User`. `[AllowAnonymous]` simply tells the authorization middleware not to reject unauthenticated requests. You can have an `[AllowAnonymous]` endpoint that still checks `User.Identity.IsAuthenticated` to show different content for logged-in vs. anonymous users.

> [!summary] Section Summary
> `[AllowAnonymous]` overrides authorization requirements to allow unauthenticated access. It is essential when using a global fallback policy. It does not disable authentication -- it only tells authorization to allow the request through regardless of authentication status.
