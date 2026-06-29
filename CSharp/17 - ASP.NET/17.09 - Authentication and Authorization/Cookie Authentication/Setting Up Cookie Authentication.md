---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


Cookie authentication is configured in two places: service registration in `Program.cs` and middleware placement in the request pipeline.

### Service Registration

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
        options.LogoutPath = "/Account/Logout";
        options.AccessDeniedPath = "/Account/AccessDenied";
        options.ExpireTimeSpan = TimeSpan.FromHours(2);
        options.SlidingExpiration = true;
    });

builder.Services.AddControllersWithViews();

var app = builder.Build();
```

### Middleware Registration

```csharp
// Order matters -- Authentication must come before Authorization
app.UseRouting();

app.UseAuthentication();  // Reads the cookie and sets HttpContext.User
app.UseAuthorization();   // Checks if the user is allowed to access the resource

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### Explanation of Each Option

- **`CookieAuthenticationDefaults.AuthenticationScheme`** -- This is the string constant `"Cookies"`. It sets cookie authentication as the default scheme so you do not have to specify it every time you call `SignInAsync` or use `[Authorize]`.

- **`LoginPath`** -- When an unauthenticated user tries to access a protected resource, the middleware redirects them to this path. The original URL is preserved in the `ReturnUrl` query parameter so the user can be redirected back after login.

- **`LogoutPath`** -- The path to handle logout POST requests. Used internally by the authentication system.

- **`AccessDeniedPath`** -- When an authenticated user tries to access a resource they are not authorized for (e.g., they lack the required role), they are redirected here instead of the login page.

- **`ExpireTimeSpan`** -- How long the cookie remains valid after it is issued. After this time, the cookie is considered expired and the user must log in again. This is independent of the browser cookie expiration.

- **`SlidingExpiration`** -- When `true`, the cookie's expiration is reset on every request, as long as more than half of the `ExpireTimeSpan` has elapsed. This keeps active users logged in while still expiring idle sessions.

> [!danger] Security Warning
> Always place `UseAuthentication()` **before** `UseAuthorization()` in the middleware pipeline. If you reverse the order, authorization checks will run before the user's identity is established, and every request will appear unauthenticated.

> [!summary] Section Summary
> Register cookie auth with `AddAuthentication().AddCookie()` in services, then add `UseAuthentication()` and `UseAuthorization()` middleware in the correct order.
