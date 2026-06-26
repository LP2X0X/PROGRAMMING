---
tags: [csharp, asp-net-core, authentication, cookies, security]
aliases: [Cookie Auth, Cookie-Based Authentication, ASP.NET Core Cookies]
status: complete
date: 2026-06-18
---

# Cookie Authentication

## Table of Contents

- [[#What Cookie Authentication Is]]
- [[#How Cookie Authentication Works]]
- [[#Setting Up Cookie Authentication]]
- [[#Cookie Options Deep Dive]]
- [[#Signing In]]
- [[#Signing Out]]
- [[#Building the ClaimsPrincipal Manually]]
- [[#Full Login and Logout Flow]]
- [[#Cookie Security]]
- [[#Persistent vs Session Cookies]]
- [[#Cookie Encryption and Data Protection]]
- [[#Anti-Forgery Tokens]]
- [[#Cookie Authentication Events]]
- [[#Complete Real-World Example]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]
- [[#Further Reading]]

---

## What Cookie Authentication Is

> [!info] Definition
> **Cookie authentication** is an authentication scheme where the server issues an encrypted cookie after a successful login. The browser automatically stores this cookie and sends it back with every subsequent HTTP request, allowing the server to identify the user without requiring them to re-enter credentials.

Cookie authentication is the **default and recommended** authentication scheme for server-rendered web applications built with ASP.NET Core MVC or Razor Pages. Unlike token-based schemes (such as [[JWT Authentication]]), cookie authentication relies on the browser's built-in cookie storage and automatic cookie transmission behavior.

The core idea is straightforward:

1. The user proves their identity once (typically via a login form).
2. The server packages the user's identity information (claims) into an encrypted, tamper-proof cookie.
3. The browser stores this cookie and attaches it to every request to the same domain.
4. The server decrypts the cookie on each request to reconstruct the user's identity.

This approach works naturally with the stateless nature of HTTP -- each request carries everything the server needs to know about the user, encoded inside the cookie.

> [!warning] Common Misconception
> Cookie authentication does **not** store the user's password in the cookie. The cookie contains an encrypted version of the user's **claims** (name, roles, email, etc.) -- never raw credentials. Even if someone intercepted the cookie, they would not be able to extract the password from it.

> [!summary] Section Summary
> Cookie authentication is the standard scheme for server-rendered ASP.NET Core apps. The server encrypts user identity data into a cookie after login, and the browser sends it back automatically on every request.

---

## How Cookie Authentication Works

The full cookie authentication flow involves a handshake between the browser, the server, and the authentication middleware. Here is the complete sequence:

### Step-by-Step Flow

**1. User submits login form**

The user navigates to a login page and submits their email/username and password via an HTML form. This is a standard HTTP POST request.

**2. Server validates credentials**

The server receives the POST request and checks the submitted credentials against a data store -- this could be a database table, [[ASP.NET Core Identity]], an external identity provider, or any custom validation logic.

**3. Server creates a ClaimsPrincipal**

If the credentials are valid, the server constructs a `ClaimsPrincipal` object. This object holds the user's **claims** -- pieces of information about the user such as their name, email, roles, and any custom data.

**4. Server calls `HttpContext.SignInAsync()`**

The server calls `HttpContext.SignInAsync()`, passing the `ClaimsPrincipal`. The cookie authentication handler takes over:
- It serializes the `ClaimsPrincipal` into bytes.
- It encrypts those bytes using the ASP.NET Core Data Protection system.
- It creates an HTTP `Set-Cookie` response header containing the encrypted payload.

**5. Browser stores the cookie**

The browser receives the `Set-Cookie` header and stores the cookie according to its attributes (domain, path, expiration, etc.).

**6. Subsequent requests include the cookie**

On every subsequent request to the same domain, the browser automatically attaches the authentication cookie in the `Cookie` request header. The user does not need to do anything -- this is built-in browser behavior.

**7. Middleware decrypts the cookie and sets `HttpContext.User`**

The cookie authentication middleware runs early in the ASP.NET Core request pipeline. It:
- Reads the cookie from the incoming request.
- Decrypts and deserializes it back into a `ClaimsPrincipal`.
- Sets `HttpContext.User` to that principal.

From this point onward, any code in the pipeline (controllers, Razor Pages, authorization filters) can access `HttpContext.User` to determine who the user is and what they are authorized to do.

> [!tip] Key Insight
> The cookie authentication middleware does **not** contact the database on every request. It reconstructs the user's identity entirely from the encrypted cookie. This makes cookie auth very fast -- but it also means that if the user's roles or permissions change in the database, the cookie will still contain the old data until it expires or the user re-logs in. You can use `OnValidatePrincipal` events to handle this (covered in the [[#Cookie Authentication Events]] section).

> [!summary] Section Summary
> The flow is: submit credentials, validate them, build a ClaimsPrincipal, encrypt it into a cookie via `SignInAsync()`, then on every subsequent request the middleware decrypts the cookie and populates `HttpContext.User`.

---

## Setting Up Cookie Authentication

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

---

## Cookie Options Deep Dive

The `CookieAuthenticationOptions` class provides extensive control over how cookies are created, transmitted, and validated.

### Options Reference Table

| Option | Type | Default | Description |
|---|---|---|---|
| `LoginPath` | `PathString` | `/Account/Login` | Redirect path for unauthenticated users |
| `LogoutPath` | `PathString` | `/Account/Logout` | Path for logout handling |
| `AccessDeniedPath` | `PathString` | `/Account/AccessDenied` | Redirect path for unauthorized users |
| `ExpireTimeSpan` | `TimeSpan` | 14 days | How long the authentication ticket is valid |
| `SlidingExpiration` | `bool` | `true` | Whether to renew expiration on active requests |
| `ReturnUrlParameter` | `string` | `"ReturnUrl"` | Query string parameter name for the return URL |
| `Cookie.Name` | `string` | `.AspNetCore.Cookies` | The name of the cookie |
| `Cookie.HttpOnly` | `bool` | `true` | If `true`, cookie is inaccessible to JavaScript |
| `Cookie.SecurePolicy` | `CookieSecurePolicy` | `SameAsRequest` | When to send the cookie over HTTPS only |
| `Cookie.SameSite` | `SameSiteMode` | `Lax` | CSRF protection mode |
| `Cookie.Domain` | `string` | (not set) | Domain the cookie is valid for |
| `Cookie.Path` | `string` | `/` | Path the cookie is valid for |
| `Cookie.MaxAge` | `TimeSpan?` | `null` | Browser-level max age of the cookie |

### Detailed Cookie Properties

```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        // Redirect paths
        options.LoginPath = "/Account/Login";
        options.LogoutPath = "/Account/Logout";
        options.AccessDeniedPath = "/Account/AccessDenied";
        options.ReturnUrlParameter = "returnUrl";

        // Ticket lifetime
        options.ExpireTimeSpan = TimeSpan.FromHours(2);
        options.SlidingExpiration = true;

        // Cookie properties
        options.Cookie.Name = "MyApp.Auth";
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.SameSite = SameSiteMode.Strict;
        options.Cookie.Domain = ".myapp.com";
        options.Cookie.Path = "/";
        options.Cookie.MaxAge = TimeSpan.FromHours(2);
    });
```

> [!warning] Common Misconception
> `ExpireTimeSpan` and `Cookie.MaxAge` are **not** the same thing. `ExpireTimeSpan` controls the authentication ticket validity inside the encrypted cookie. `Cookie.MaxAge` controls how long the browser keeps the cookie. If the browser cookie has not expired but the ticket inside it has, the user will still need to log in again. Best practice is to keep them aligned or rely on `ExpireTimeSpan` alone.

> [!tip] Production Best Practice
> In production, always set `Cookie.SecurePolicy = CookieSecurePolicy.Always` and `Cookie.SameSite = SameSiteMode.Strict` (or at minimum `Lax`). These settings prevent the cookie from being sent over insecure connections and provide protection against CSRF attacks.

> [!summary] Section Summary
> Cookie options control redirect paths, ticket lifetime, sliding expiration, and the physical cookie properties (name, security flags, SameSite mode). Configure these carefully for production security.

---

## Signing In

The sign-in process involves creating the user's identity and calling `HttpContext.SignInAsync()` to issue the cookie.

### The SignInAsync Method

```csharp
await HttpContext.SignInAsync(
    scheme: CookieAuthenticationDefaults.AuthenticationScheme,
    principal: claimsPrincipal,
    properties: new AuthenticationProperties
    {
        IsPersistent = true,               // Persistent cookie (survives browser close)
        ExpiresUtc = DateTimeOffset.UtcNow.AddHours(2),
        AllowRefresh = true                // Allow sliding expiration
    }
);
```

### Complete Sign-In Process

```csharp
// 1. Build the claims
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.DisplayName),
    new Claim(ClaimTypes.Email, user.Email),
    new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
    new Claim(ClaimTypes.Role, "Admin"),
    new Claim("Department", "Engineering")  // Custom claim
};

// 2. Create an identity from those claims
var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);

// 3. Create a principal from the identity
var principal = new ClaimsPrincipal(identity);

// 4. Sign in -- this encrypts the principal into a cookie
await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    principal,
    new AuthenticationProperties
    {
        IsPersistent = rememberMe
    }
);
```

> [!ad-note]
> The second parameter of `ClaimsIdentity` constructor -- the `authenticationType` string -- is critical. If you pass `null` or omit it, the identity's `IsAuthenticated` property will return `false`, and the user will appear unauthenticated even though they have a valid cookie. Always pass the authentication scheme name or any non-null string.

> [!summary] Section Summary
> Sign-in requires building a claims list, creating a `ClaimsIdentity` with an authentication type, wrapping it in a `ClaimsPrincipal`, and calling `HttpContext.SignInAsync()`.

---

## Signing Out

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

---

## Building the ClaimsPrincipal Manually

Understanding how to construct a `ClaimsPrincipal` from scratch is fundamental to cookie authentication. Here is a detailed breakdown of each component.

### Claims

A **claim** is a name-value pair representing a fact about the user. ASP.NET Core defines standard claim types in the `ClaimTypes` class.

```csharp
var claims = new List<Claim>
{
    // Standard claims
    new Claim(ClaimTypes.NameIdentifier, "12345"),
    new Claim(ClaimTypes.Name, "john.doe"),
    new Claim(ClaimTypes.Email, "john@example.com"),
    new Claim(ClaimTypes.GivenName, "John"),
    new Claim(ClaimTypes.Surname, "Doe"),
    new Claim(ClaimTypes.Role, "Admin"),
    new Claim(ClaimTypes.Role, "Manager"),  // Multiple roles are fine

    // Custom claims
    new Claim("Department", "Engineering"),
    new Claim("EmployeeId", "EMP-42"),
    new Claim("Tier", "Premium")
};
```

### ClaimsIdentity

A `ClaimsIdentity` bundles claims together with an authentication type. Think of it like an ID card -- it contains information and states what authority issued it.

```csharp
var identity = new ClaimsIdentity(
    claims: claims,
    authenticationType: CookieAuthenticationDefaults.AuthenticationScheme
);

// You can also specify which claim types map to Name and Role
var identity = new ClaimsIdentity(
    claims: claims,
    authenticationType: CookieAuthenticationDefaults.AuthenticationScheme,
    nameType: ClaimTypes.Name,
    roleType: ClaimTypes.Role
);
```

> [!info] Definition
> The `authenticationType` parameter determines the value of `ClaimsIdentity.IsAuthenticated`. If it is `null` or empty, `IsAuthenticated` returns `false`. If it is any non-null, non-empty string, `IsAuthenticated` returns `true`.

### ClaimsPrincipal

A `ClaimsPrincipal` represents the user and can hold multiple identities (for scenarios where the user authenticates through more than one scheme).

```csharp
var principal = new ClaimsPrincipal(identity);

// After sign-in, access claims anywhere via HttpContext:
var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
var userName = User.Identity?.Name;
var isAdmin = User.IsInRole("Admin");
var email = User.FindFirst(ClaimTypes.Email)?.Value;
```

> [!summary] Section Summary
> Build claims as a list of `Claim` objects, wrap them in a `ClaimsIdentity` with an authentication type, then wrap that in a `ClaimsPrincipal`. The principal represents the authenticated user throughout the request.

---

## Full Login and Logout Flow

Here is a complete, real-world implementation of cookie authentication including the view model, controller actions, and important security considerations.

### LoginViewModel

```csharp
public class LoginViewModel
{
    [Required(ErrorMessage = "Email is required.")]
    [EmailAddress(ErrorMessage = "Invalid email format.")]
    public string Email { get; set; } = string.Empty;

    [Required(ErrorMessage = "Password is required.")]
    [DataType(DataType.Password)]
    public string Password { get; set; } = string.Empty;

    [Display(Name = "Remember me")]
    public bool RememberMe { get; set; }
}
```

### AccountController

```csharp
public class AccountController : Controller
{
    private readonly IUserService _userService;

    public AccountController(IUserService userService)
    {
        _userService = userService;
    }

    // GET: /Account/Login
    [HttpGet]
    public IActionResult Login(string? returnUrl = null)
    {
        // If the user is already authenticated, redirect to home
        if (User.Identity?.IsAuthenticated == true)
        {
            return RedirectToAction("Index", "Home");
        }

        ViewData["ReturnUrl"] = returnUrl;
        return View();
    }

    // POST: /Account/Login
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Login(LoginViewModel model, string? returnUrl = null)
    {
        ViewData["ReturnUrl"] = returnUrl;

        if (!ModelState.IsValid)
        {
            return View(model);
        }

        // Validate credentials against the database
        var user = await _userService.ValidateCredentialsAsync(model.Email, model.Password);

        if (user is null)
        {
            ModelState.AddModelError(string.Empty, "Invalid email or password.");
            return View(model);
        }

        // Build the claims
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Name, user.DisplayName),
            new Claim(ClaimTypes.Email, user.Email)
        };

        // Add role claims
        foreach (var role in user.Roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role));
        }

        var identity = new ClaimsIdentity(
            claims,
            CookieAuthenticationDefaults.AuthenticationScheme);

        var principal = new ClaimsPrincipal(identity);

        // Sign in
        await HttpContext.SignInAsync(
            CookieAuthenticationDefaults.AuthenticationScheme,
            principal,
            new AuthenticationProperties
            {
                IsPersistent = model.RememberMe,
                ExpiresUtc = model.RememberMe
                    ? DateTimeOffset.UtcNow.AddDays(30)
                    : DateTimeOffset.UtcNow.AddHours(2)
            });

        // Redirect to return URL or home
        if (!string.IsNullOrEmpty(returnUrl) && Url.IsLocalUrl(returnUrl))
        {
            return Redirect(returnUrl);
        }

        return RedirectToAction("Index", "Home");
    }

    // POST: /Account/Logout
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Logout()
    {
        await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
        return RedirectToAction("Index", "Home");
    }

    // GET: /Account/AccessDenied
    [HttpGet]
    public IActionResult AccessDenied()
    {
        return View();
    }
}
```

> [!danger] Security Warning
> Always validate that `returnUrl` is a local URL using `Url.IsLocalUrl(returnUrl)` before redirecting. Without this check, an attacker could craft a link like `/Account/Login?returnUrl=https://evil.com` and redirect the user to a malicious site after login -- this is called an **open redirect** attack.

> [!warning] Common Misconception
> Never return different error messages for "user not found" vs "wrong password." An attacker could use this to enumerate valid email addresses in your system. Always use a generic message like "Invalid email or password."

> [!summary] Section Summary
> The full flow includes a `LoginViewModel` with validation, a GET action to display the form, a POST action that validates credentials and signs in, and a POST logout action. Always protect against open redirect and credential enumeration attacks.

---

## Cookie Security

Cookie security is critical because the authentication cookie is the key to the user's session. If an attacker can steal or forge it, they can impersonate the user.

### HttpOnly

```csharp
options.Cookie.HttpOnly = true;  // This is the default
```

When `HttpOnly` is `true`, the cookie is inaccessible to JavaScript via `document.cookie`. This protects against **Cross-Site Scripting (XSS)** attacks. Even if an attacker injects malicious JavaScript into your page, that script cannot read or exfiltrate the authentication cookie.

> [!danger] Security Warning
> Never set `HttpOnly` to `false` for authentication cookies. There is no legitimate reason for client-side JavaScript to access the auth cookie. If you need user data on the client, expose it through a separate API endpoint instead.

### Secure

```csharp
options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
```

When set to `Always`, the cookie is only sent over HTTPS connections. This prevents the cookie from being transmitted in plaintext over HTTP, protecting against **man-in-the-middle (MITM)** attacks where an attacker on the same network could intercept the cookie.

| Value | Behavior |
|---|---|
| `Always` | Cookie only sent over HTTPS |
| `SameAsRequest` | Cookie sent over the same protocol as the request |
| `None` | Cookie sent over both HTTP and HTTPS |

### SameSite

```csharp
options.Cookie.SameSite = SameSiteMode.Strict;
```

The `SameSite` attribute controls whether the cookie is sent with cross-site requests, providing protection against **Cross-Site Request Forgery (CSRF)** attacks.

| Mode | Behavior | Use Case |
|---|---|---|
| `Strict` | Cookie never sent on cross-site requests | Maximum security; may break external links |
| `Lax` | Cookie sent on top-level navigations (GET) but not on cross-site POST | Good balance of security and usability |
| `None` | Cookie always sent (requires `Secure = true`) | Only for cross-site scenarios (OAuth, embedded iframes) |

> [!example] CSRF Attack Scenario
> Without `SameSite` protection, an attacker's website could include:
> ```html
> <form action="https://yourbank.com/transfer" method="POST">
>     <input type="hidden" name="amount" value="10000" />
>     <input type="hidden" name="to" value="attacker-account" />
> </form>
> <script>document.forms[0].submit();</script>
> ```
> When the victim visits the attacker's page, their browser would automatically attach the bank's auth cookie to this cross-site POST. With `SameSite = Strict` or `Lax`, the browser refuses to attach the cookie to this cross-origin request.

> [!tip] Recommended Security Configuration
> For most server-rendered applications, use this combination:
> ```csharp
> options.Cookie.HttpOnly = true;
> options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
> options.Cookie.SameSite = SameSiteMode.Lax;
> ```
> Use `Strict` if your app is never linked to from external sites. Use `Lax` if users might follow links to your app from emails or other sites.

> [!summary] Section Summary
> `HttpOnly` prevents XSS cookie theft, `Secure` prevents MITM interception, and `SameSite` prevents CSRF attacks. Together, these three flags form the security foundation for authentication cookies.

---

## Persistent vs Session Cookies

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

---

## Cookie Encryption and Data Protection

ASP.NET Core encrypts authentication cookies using the **Data Protection** system. Understanding this is essential for production deployments, especially in web farm or containerized environments.

### How It Works

When you call `SignInAsync()`:

1. The `ClaimsPrincipal` is serialized into a byte array.
2. The Data Protection system encrypts and signs the byte array using a key from the key ring.
3. The encrypted bytes are Base64-encoded and set as the cookie value.

When the cookie comes back on a request:

1. The middleware reads the cookie value.
2. The Data Protection system decrypts and verifies the signature.
3. The bytes are deserialized back into a `ClaimsPrincipal`.

### Key Storage

By default, Data Protection keys are stored at:

- **Windows**: `%LOCALAPPDATA%\ASP.NET\DataProtection-Keys`
- **Linux/macOS**: `~/.aspnet/DataProtection-Keys`

> [!danger] Security Warning -- Web Farms
> In a web farm (multiple servers behind a load balancer), each server has its own key ring by default. A cookie encrypted by Server A cannot be decrypted by Server B. This causes users to be randomly logged out when their requests hit different servers. You must configure **shared key storage** for web farms.

### Configuring Shared Key Storage

```csharp
// Store keys in a shared location for web farm scenarios
builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\keys"))
    .SetApplicationName("MyApp");

// Or use Azure Blob Storage
builder.Services.AddDataProtection()
    .PersistKeysToAzureBlobStorage(connectionString, containerName, blobName)
    .ProtectKeysWithAzureKeyVault(keyUri, tokenCredential);

// Or use Redis
builder.Services.AddDataProtection()
    .PersistKeysToStackExchangeRedis(connectionMultiplexer, "DataProtection-Keys");
```

> [!tip] SetApplicationName
> When multiple applications share the same key storage, call `SetApplicationName("MyApp")` to isolate each app's keys. Without this, one app could accidentally decrypt another app's cookies.

> [!summary] Section Summary
> Cookies are encrypted using ASP.NET Core Data Protection. Keys are stored locally by default. For web farms, configure shared key storage (file system, database, Redis, or cloud storage) so all servers can decrypt the same cookies.

---

## Anti-Forgery Tokens

Anti-forgery tokens (also called CSRF tokens) are a complementary security mechanism that works alongside cookie authentication to protect against **Cross-Site Request Forgery** attacks.

### The CSRF Problem

When a user is authenticated via cookies, the browser automatically sends those cookies with every request to your domain -- including requests initiated by malicious sites. An attacker can exploit this by tricking the user's browser into sending a request to your server.

### How Anti-Forgery Tokens Work

1. The server generates a unique token and embeds it in the HTML form (as a hidden field) and in a separate cookie.
2. When the form is submitted, the server compares the token from the form field with the token from the cookie.
3. A cross-site attacker cannot read the token from your page (same-origin policy), so they cannot include the correct token in their forged request.

### Usage in Razor Views

```csharp
<!-- The asp-antiforgery tag helper automatically includes the token -->
<form asp-action="Login" asp-controller="Account" method="post">
    <!-- Hidden __RequestVerificationToken field is auto-generated -->
    <input asp-for="Email" />
    <input asp-for="Password" type="password" />
    <button type="submit">Log In</button>
</form>

<!-- Or manually add the token -->
<form method="post" action="/Account/Login">
    @Html.AntiForgeryToken()
    <!-- form fields -->
</form>
```

### Usage in Controllers

```csharp
[HttpPost]
[ValidateAntiForgeryToken]  // Validates the token on POST
public async Task<IActionResult> Login(LoginViewModel model)
{
    // Action code here
}
```

### Global Anti-Forgery Configuration

```csharp
// Apply anti-forgery validation to all POST actions globally
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute());
});
```

> [!example] Why Anti-Forgery Tokens Are Needed Even with SameSite Cookies
> `SameSite` cookies are a strong defense against CSRF, but they are not perfect:
> - Older browsers may not support `SameSite`.
> - `Lax` mode still allows cookies on top-level GET navigations.
> - Defense in depth -- using both `SameSite` and anti-forgery tokens provides layered protection.

> [!summary] Section Summary
> Anti-forgery tokens prevent CSRF attacks by requiring a server-generated token in both a cookie and a hidden form field. Always use `[ValidateAntiForgeryToken]` on POST actions or apply `AutoValidateAntiforgeryTokenAttribute` globally.

---

## Cookie Authentication Events

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

---

## Complete Real-World Example

Here is a full, production-ready implementation of cookie authentication, from `Program.cs` through the controller and view.

### Program.cs

```csharp
using Microsoft.AspNetCore.Authentication.Cookies;
using System.Security.Claims;

var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute());
});

builder.Services.AddScoped<IUserService, UserService>();

// Configure cookie authentication
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
        options.LogoutPath = "/Account/Logout";
        options.AccessDeniedPath = "/Account/AccessDenied";
        options.ExpireTimeSpan = TimeSpan.FromHours(2);
        options.SlidingExpiration = true;

        options.Cookie.Name = "MyApp.Auth";
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.SameSite = SameSiteMode.Lax;

        options.Events = new CookieAuthenticationEvents
        {
            OnValidatePrincipal = async context =>
            {
                var userService = context.HttpContext.RequestServices
                    .GetRequiredService<IUserService>();

                var userId = context.Principal?
                    .FindFirst(ClaimTypes.NameIdentifier)?.Value;

                if (userId is null)
                {
                    context.RejectPrincipal();
                    await context.HttpContext.SignOutAsync(
                        CookieAuthenticationDefaults.AuthenticationScheme);
                    return;
                }

                var isActive = await userService.IsUserActiveAsync(int.Parse(userId));
                if (!isActive)
                {
                    context.RejectPrincipal();
                    await context.HttpContext.SignOutAsync(
                        CookieAuthenticationDefaults.AuthenticationScheme);
                }
            }
        };
    });

builder.Services.AddAuthorization();

var app = builder.Build();

// Middleware pipeline
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### AccountController

```csharp
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

public class AccountController : Controller
{
    private readonly IUserService _userService;

    public AccountController(IUserService userService)
    {
        _userService = userService;
    }

    [HttpGet]
    [AllowAnonymous]
    public IActionResult Login(string? returnUrl = null)
    {
        if (User.Identity?.IsAuthenticated == true)
        {
            return RedirectToAction("Index", "Home");
        }

        ViewData["ReturnUrl"] = returnUrl;
        return View();
    }

    [HttpPost]
    [AllowAnonymous]
    public async Task<IActionResult> Login(LoginViewModel model, string? returnUrl = null)
    {
        ViewData["ReturnUrl"] = returnUrl;

        if (!ModelState.IsValid)
        {
            return View(model);
        }

        var user = await _userService.ValidateCredentialsAsync(
            model.Email, model.Password);

        if (user is null)
        {
            ModelState.AddModelError(string.Empty, "Invalid email or password.");
            return View(model);
        }

        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Name, user.DisplayName),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim("SecurityStamp", user.SecurityStamp)
        };

        foreach (var role in user.Roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role));
        }

        var identity = new ClaimsIdentity(
            claims,
            CookieAuthenticationDefaults.AuthenticationScheme);

        var principal = new ClaimsPrincipal(identity);

        await HttpContext.SignInAsync(
            CookieAuthenticationDefaults.AuthenticationScheme,
            principal,
            new AuthenticationProperties
            {
                IsPersistent = model.RememberMe,
                ExpiresUtc = model.RememberMe
                    ? DateTimeOffset.UtcNow.AddDays(30)
                    : null
            });

        if (!string.IsNullOrEmpty(returnUrl) && Url.IsLocalUrl(returnUrl))
        {
            return Redirect(returnUrl);
        }

        return RedirectToAction("Index", "Home");
    }

    [HttpPost]
    [Authorize]
    public async Task<IActionResult> Logout()
    {
        await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
        return RedirectToAction("Index", "Home");
    }

    [HttpGet]
    [AllowAnonymous]
    public IActionResult AccessDenied()
    {
        return View();
    }
}
```

### Login.cshtml (Razor View)

```csharp
@model LoginViewModel

@{
    ViewData["Title"] = "Log In";
}

<div class="row justify-content-center">
    <div class="col-md-6">
        <h2>Log In</h2>

        <div asp-validation-summary="ModelOnly" class="text-danger"></div>

        <form asp-action="Login" asp-controller="Account" method="post">
            <input type="hidden" name="returnUrl" value="@ViewData["ReturnUrl"]" />

            <div class="mb-3">
                <label asp-for="Email" class="form-label"></label>
                <input asp-for="Email" class="form-control" autocomplete="email" />
                <span asp-validation-for="Email" class="text-danger"></span>
            </div>

            <div class="mb-3">
                <label asp-for="Password" class="form-label"></label>
                <input asp-for="Password" class="form-control" autocomplete="current-password" />
                <span asp-validation-for="Password" class="text-danger"></span>
            </div>

            <div class="mb-3 form-check">
                <input asp-for="RememberMe" class="form-check-input" />
                <label asp-for="RememberMe" class="form-check-label"></label>
            </div>

            <button type="submit" class="btn btn-primary w-100">Log In</button>
        </form>
    </div>
</div>
```

### Logout Partial (for navbar)

```csharp
@if (User.Identity?.IsAuthenticated == true)
{
    <form asp-action="Logout" asp-controller="Account" method="post" class="d-inline">
        <span class="navbar-text me-2">Hello, @User.Identity.Name</span>
        <button type="submit" class="btn btn-outline-secondary btn-sm">Log Out</button>
    </form>
}
else
{
    <a asp-action="Login" asp-controller="Account" class="btn btn-primary btn-sm">Log In</a>
}
```

> [!tip] Complete Working Setup
> To protect specific controllers or actions, use `[Authorize]`:
> ```csharp
> [Authorize]  // All actions require authentication
> public class DashboardController : Controller { }
>
> [Authorize(Roles = "Admin")]  // Requires Admin role
> public class AdminController : Controller { }
> ```

> [!summary] Section Summary
> A production-ready cookie authentication setup includes: Program.cs with services and middleware, an AccountController with Login/Logout actions, a Razor login form with anti-forgery protection, and `[Authorize]` attributes on protected resources.

---

## Comprehensive Summary

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

---

## Related Topics

- [[Authentication Overview]] -- high-level view of authentication in ASP.NET Core
- [[JWT Authentication]] -- token-based authentication for SPAs and APIs
- [[ASP.NET Core Identity]] -- full-featured identity framework built on top of cookie auth
- [[Authorization Policies]] -- policy-based authorization for fine-grained access control
- [[Middleware Overview]] -- understanding the ASP.NET Core request pipeline

---

## Further Reading

- [Microsoft Docs -- Use cookie authentication without ASP.NET Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie)
- [Microsoft Docs -- ASP.NET Core Data Protection](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/introduction)
- [Microsoft Docs -- Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks](https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery)
- [OWASP -- Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP -- Cross-Site Request Forgery Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [RFC 6265 -- HTTP State Management Mechanism (Cookies)](https://www.rfc-editor.org/rfc/rfc6265)
