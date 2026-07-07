---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


The `[Authorize]` and `[AllowAnonymous]` attributes interact in a specific way that is important to understand.

### Controller-Level Authorize, Action-Level AllowAnonymous

```csharp
[Authorize]
public class AccountController : Controller
{
    // Requires authentication (inherits from controller)
    public IActionResult Profile() => View();

    // Requires authentication (inherits from controller)
    public IActionResult Settings() => View();

    // Allows anonymous access -- overrides the controller-level [Authorize]
    [AllowAnonymous]
    public IActionResult Login() => View();

    // Allows anonymous access
    [AllowAnonymous]
    public IActionResult Register() => View();
}
```

> [!warning] Common Misconception
> `[AllowAnonymous]` **always wins** when it conflicts with `[Authorize]`. Even if you apply a policy-based `[Authorize]` at the controller level, an `[AllowAnonymous]` on a specific action will bypass all authorization checks for that action. This is by design -- it lets you create login pages on otherwise protected controllers.

### Global Authorization with Selective Anonymous Access

You can require authorization globally in `Program.cs` and then opt out individual endpoints:

```csharp
builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});
```

With this configuration, **every endpoint** requires authentication by default. Use `[AllowAnonymous]` on specific controllers or actions that should be public.

> [!tip]
> Setting a `FallbackPolicy` is a safer default than relying on developers remembering to add `[Authorize]` to every controller. With a fallback policy, forgotten attributes result in *over-protection* (blocking access) rather than *under-protection* (leaking data).

> [!summary] Section Summary
> `[AllowAnonymous]` overrides `[Authorize]` whenever they conflict. Use `FallbackPolicy` to require authentication globally and opt out with `[AllowAnonymous]` for a secure-by-default approach.
