---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


The `IsPersistent` property in `AuthenticationProperties` determines whether the cookie survives when the user closes their browser.

### Session Cookies

By default, when `IsPersistent` is `false` (or not set), the cookie is a **session cookie**. The browser deletes it when the user closes the browser. This is the more secure default because it limits the window of exposure.

```csharp
await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    principal,
    new AuthenticationProperties
    {
        IsPersistent = false  // Session cookie -- deleted on browser close
    });
```

### Persistent Cookies (Remember Me)

When `IsPersistent` is `true`, the cookie is written to disk by the browser and survives browser restarts. This is how "Remember Me" functionality works.

```csharp
await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    principal,
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTimeOffset.UtcNow.AddDays(30)  // Persistent for 30 days
    });
```

### Implementing "Remember Me"

```csharp
// In the login POST action:
var authProperties = new AuthenticationProperties
{
    IsPersistent = model.RememberMe,
    ExpiresUtc = model.RememberMe
        ? DateTimeOffset.UtcNow.AddDays(30)   // Long-lived if "Remember Me"
        : null                                  // Use default ExpireTimeSpan
};

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    principal,
    authProperties);
```

> [!warning] Common Misconception
> Setting `IsPersistent = true` without specifying `ExpiresUtc` will use the `ExpireTimeSpan` from the cookie options. If `ExpireTimeSpan` is set to 2 hours, the persistent cookie will still expire after 2 hours -- it will just survive browser restarts during that period.

> [!summary] Section Summary
> Session cookies are deleted when the browser closes. Persistent cookies (`IsPersistent = true`) survive browser restarts and are used for "Remember Me". Always set an explicit `ExpiresUtc` for persistent cookies.
