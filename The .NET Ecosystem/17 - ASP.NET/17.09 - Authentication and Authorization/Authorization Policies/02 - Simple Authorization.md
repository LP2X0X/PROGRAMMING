---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


The simplest form of authorization in ASP.NET Core is the `[Authorize]` attribute with no parameters. It does not check roles, claims, or policies -- it simply requires that the user is authenticated.

```csharp
[Authorize]
public class DashboardController : Controller
{
    // All actions in this controller require authentication.
    // Any logged-in user can access these actions.

    public IActionResult Index()
    {
        return View();
    }

    public IActionResult Settings()
    {
        return View();
    }
}
```

You can also apply `[Authorize]` to individual actions rather than the entire controller:

```csharp
public class HomeController : Controller
{
    public IActionResult Index()
    {
        // Anyone can access this -- no [Authorize] attribute
        return View();
    }

    [Authorize]
    public IActionResult Profile()
    {
        // Only authenticated users can access this
        return View();
    }
}
```

> [!warning] Common Misconception
> `[Authorize]` with no parameters does **not** mean "no authorization." It means "require authentication." An unauthenticated user will be redirected to the login page (for cookie auth) or receive a `401 Unauthorized` response (for bearer token auth).

> [!summary] Section Summary
> The bare `[Authorize]` attribute is the simplest authorization mechanism. It only checks that the user is authenticated -- it does not evaluate any roles, claims, or policies.
