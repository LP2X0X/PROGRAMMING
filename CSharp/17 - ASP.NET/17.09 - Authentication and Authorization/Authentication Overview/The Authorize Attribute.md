---
tags:
  - csharp
  - asp-net-core
  - authentication
  - security
---


> [!info] Definition
> The `[Authorize]` attribute marks a controller or action as requiring an authenticated user. It can optionally specify roles, policies, or authentication schemes.

### Basic Usage

```csharp
// Require authentication for all actions in this controller
[Authorize]
public class AccountController : Controller
{
    public IActionResult Profile() => View();
    public IActionResult Settings() => View();
}
```

```csharp
// Require authentication for a specific action only
public class HomeController : Controller
{
    public IActionResult Index() => View(); // Public

    [Authorize]
    public IActionResult Dashboard() => View(); // Requires auth
}
```

### Role-Based Authorization

```csharp
[Authorize(Roles = "Admin")]
public class AdminController : Controller
{
    public IActionResult ManageUsers() => View();
}

// Multiple roles -- user must have at least ONE of them
[Authorize(Roles = "Admin,Manager")]
public IActionResult Reports() => View();

// Stacking -- user must have ALL specified roles
[Authorize(Roles = "Admin")]
[Authorize(Roles = "Manager")]
public IActionResult SensitiveOperation() => View();
```

### Policy-Based Authorization

```csharp
[Authorize(Policy = "RequireAdminRole")]
public IActionResult AdminOnly() => View();

[Authorize(Policy = "MinimumAge18")]
public IActionResult AdultContent() => View();
```

### Global Authorization

To require authentication for all endpoints by default:

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});
```

> [!tip]
> Setting a `FallbackPolicy` that requires authentication means you do not need to put `[Authorize]` on every controller. Instead, you use `[AllowAnonymous]` on the few endpoints that should be public (login page, registration, health checks).

> [!summary] Section Summary
> The `[Authorize]` attribute enforces authentication at the controller or action level. It supports role-based and policy-based authorization. For applications where most endpoints require auth, set a global `FallbackPolicy` and selectively allow anonymous access.
