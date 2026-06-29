---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


Signing out deletes the authentication cookie and effectively ends the user's session.

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Logout()
{
    await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
    return RedirectToAction("Index", "Home");
}
```

`SignOutAsync` instructs the cookie authentication handler to:

1. Issue a `Set-Cookie` response header that replaces the existing cookie with an expired one.
2. The browser, upon receiving the expired cookie, removes it from its cookie store.

> [!warning] Common Misconception
> Calling `SignOutAsync` does **not** invalidate the cookie on the server side. If someone previously copied the raw cookie value, they could theoretically replay it until its `ExpireTimeSpan` elapses. To defend against this, use the `OnValidatePrincipal` event to check a security stamp or session identifier stored in the database on every request.

> [!tip] Always Use POST for Logout
> Logout should always be a POST request protected by `[ValidateAntiForgeryToken]`. Using a GET request for logout makes your app vulnerable to CSRF attacks where a malicious site could log users out by embedding an image tag pointing to your logout URL.

> [!summary] Section Summary
> Call `HttpContext.SignOutAsync()` to delete the authentication cookie. Always use POST with anti-forgery protection for the logout endpoint.
