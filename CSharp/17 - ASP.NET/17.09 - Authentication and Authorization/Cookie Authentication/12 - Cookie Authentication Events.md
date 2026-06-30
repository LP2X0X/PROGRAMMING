---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


The `CookieAuthenticationEvents` class provides hooks into the cookie authentication lifecycle. These are useful for custom logic like re-validating users, logging, or modifying redirects.

### Key Events

| Event | When It Fires |
|---|---|
| `OnValidatePrincipal` | Every request, after the cookie is decrypted |
| `OnSigningIn` | Just before the cookie is written |
| `OnSignedIn` | Just after the cookie is written |
| `OnSigningOut` | When `SignOutAsync` is called |
| `OnRedirectToLogin` | When redirecting an unauthenticated user to login |
| `OnRedirectToAccessDenied` | When redirecting an unauthorized user |
| `OnRedirectToReturnUrl` | After successful login, before redirecting back |

### Validating the Principal on Every Request

This is the most commonly used event. It allows you to check whether the user's account is still valid in the database on every request -- for example, if an admin disabled the account or changed the user's roles.

```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.Events = new CookieAuthenticationEvents
        {
            OnValidatePrincipal = async context =>
            {
                var userService = context.HttpContext.RequestServices
                    .GetRequiredService<IUserService>();

                var userId = context.Principal?.FindFirst(ClaimTypes.NameIdentifier)?.Value;

                if (userId is null)
                {
                    context.RejectPrincipal();
                    await context.HttpContext.SignOutAsync(
                        CookieAuthenticationDefaults.AuthenticationScheme);
                    return;
                }

                var user = await userService.GetByIdAsync(int.Parse(userId));

                // If the user no longer exists or is deactivated, reject the cookie
                if (user is null || !user.IsActive)
                {
                    context.RejectPrincipal();
                    await context.HttpContext.SignOutAsync(
                        CookieAuthenticationDefaults.AuthenticationScheme);
                    return;
                }

                // Optionally, check a security stamp to detect password changes
                var stamp = context.Principal?.FindFirst("SecurityStamp")?.Value;
                if (stamp != user.SecurityStamp)
                {
                    context.RejectPrincipal();
                    await context.HttpContext.SignOutAsync(
                        CookieAuthenticationDefaults.AuthenticationScheme);
                }
            }
        };
    });
```

### Customizing Redirect Behavior for API Calls

By default, cookie auth returns a 302 redirect to the login page. For API calls from JavaScript, you might want a 401 status code instead.

```csharp
options.Events = new CookieAuthenticationEvents
{
    OnRedirectToLogin = context =>
    {
        if (context.Request.Path.StartsWithSegments("/api"))
        {
            context.Response.StatusCode = StatusCodes.Status401Unauthorized;
        }
        else
        {
            context.Response.Redirect(context.RedirectUri);
        }
        return Task.CompletedTask;
    },
    OnRedirectToAccessDenied = context =>
    {
        if (context.Request.Path.StartsWithSegments("/api"))
        {
            context.Response.StatusCode = StatusCodes.Status403Forbidden;
        }
        else
        {
            context.Response.Redirect(context.RedirectUri);
        }
        return Task.CompletedTask;
    }
};
```

> [!tip] Performance Consideration
> `OnValidatePrincipal` fires on **every authenticated request**. Hitting the database on every request can become a performance bottleneck. Consider using a time-based check -- only validate against the database every N minutes by storing a timestamp in the claims and comparing it on each request.

> [!summary] Section Summary
> Cookie authentication events let you hook into the lifecycle for validation, logging, and custom redirect logic. `OnValidatePrincipal` is the most critical event -- use it to re-validate users against the database, but be mindful of the performance cost.
