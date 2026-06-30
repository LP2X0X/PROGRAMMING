---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


> [!tip] Complete Summary
> **Cookie authentication** is the standard authentication mechanism for server-rendered ASP.NET Core applications (MVC, Razor Pages). Here is everything covered in this note:
>
> **Core Concept**: After a user logs in, the server encrypts their identity (claims) into a cookie. The browser sends this cookie with every request, and the middleware decrypts it to populate `HttpContext.User`.
>
> **Setup**: Register with `AddAuthentication().AddCookie()` in services. Add `UseAuthentication()` before `UseAuthorization()` in middleware. Configure options like `LoginPath`, `ExpireTimeSpan`, and `SlidingExpiration`.
>
> **Sign-In Process**: Build a list of `Claim` objects, create a `ClaimsIdentity` (with a non-null `authenticationType`), wrap it in a `ClaimsPrincipal`, then call `HttpContext.SignInAsync()`.
>
> **Sign-Out**: Call `HttpContext.SignOutAsync()` to delete the cookie. Always use POST with anti-forgery protection.
>
> **Security Essentials**: Set `HttpOnly = true` (blocks XSS cookie theft), `SecurePolicy = Always` (HTTPS only), and `SameSite = Lax` or `Strict` (CSRF protection). Use `[ValidateAntiForgeryToken]` on all POST actions.
>
> **Persistent vs Session Cookies**: `IsPersistent = false` creates session cookies (deleted on browser close). `IsPersistent = true` with an `ExpiresUtc` creates "Remember Me" cookies that survive browser restarts.
>
> **Data Protection**: Cookies are encrypted using ASP.NET Core Data Protection keys. In web farm scenarios, configure shared key storage so all servers can decrypt the same cookies.
>
> **Events**: Use `OnValidatePrincipal` to re-validate users against the database on each request. Use redirect events to return 401/403 for API calls instead of redirecting to the login page.
>
> **Key Takeaway**: Cookie authentication is simple, secure, and well-suited for server-rendered apps. Combine it with proper cookie security settings, anti-forgery tokens, and principal validation events for a robust authentication system.
