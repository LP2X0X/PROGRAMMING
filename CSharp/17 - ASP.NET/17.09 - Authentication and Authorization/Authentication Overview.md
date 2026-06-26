---
tags: [csharp, asp-net-core, authentication, security]
aliases: [ASP.NET Core Authentication, Authentication Middleware, Auth Overview]
status: complete
date: 2026-06-18
---

# Authentication Overview

## Table of Contents

- [[#Authentication vs Authorization]]
- [[#The Authentication Middleware]]
- [[#Authentication Schemes]]
- [[#ClaimsPrincipal]]
- [[#Claims]]
- [[#ClaimsIdentity]]
- [[#How Authentication Works at a High Level]]
- [[#Default Scheme vs Challenge Scheme vs Forbid Scheme]]
- [[#Multiple Authentication Schemes]]
- [[#The Authorize Attribute]]
- [[#The AllowAnonymous Attribute]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]
- [[#Further Reading]]

---

## Authentication vs Authorization

> [!info] Definition
> **Authentication** answers the question: *"Who are you?"*
> **Authorization** answers the question: *"What are you allowed to do?"*

These two concepts are often confused, but they are fundamentally different steps in the security pipeline. Authentication always comes first -- you cannot determine what someone is allowed to do until you know who they are.

**Analogy:** Imagine arriving at a concert venue. Authentication is showing your ID at the door -- proving you are who you claim to be. Authorization is the staff checking your ticket to see if it grants you access to the VIP section. You might be authenticated (they know your name) but not authorized (your ticket is general admission only).

| Aspect | Authentication | Authorization |
|---|---|---|
| Question | Who are you? | What can you do? |
| Happens | First | Second |
| HTTP Status on Failure | 401 Unauthorized | 403 Forbidden |
| Middleware | `UseAuthentication()` | `UseAuthorization()` |
| Namespace | `Microsoft.AspNetCore.Authentication` | `Microsoft.AspNetCore.Authorization` |

> [!warning] Common Misconception
> The HTTP status code `401` is named "Unauthorized," but it actually means **unauthenticated**. A `403 Forbidden` is the true "unauthorized" (authenticated but lacking permission). This naming inconsistency in the HTTP spec causes endless confusion.

> [!summary] Section Summary
> Authentication identifies the user. Authorization determines their permissions. They are separate middleware components in the ASP.NET Core pipeline, and authentication must always run before authorization.

---

## The Authentication Middleware

The authentication middleware is registered by calling `app.UseAuthentication()` in your application's request pipeline. Its job is to inspect incoming requests, attempt to identify the caller, and populate `HttpContext.User` with the result.

### Middleware Ordering

The order in which middleware is registered matters enormously. The authentication middleware must sit **after routing** but **before authorization**:

```csharp
var app = builder.Build();

app.UseRouting();

app.UseAuthentication();  // WHO are you?
app.UseAuthorization();   // WHAT can you do?

app.MapControllers();
```

> [!danger] Critical
> If `UseAuthentication()` is placed after `UseAuthorization()`, authorization will run against an unauthenticated user every time. The `[Authorize]` attribute will never see a logged-in user, and every protected endpoint will return 401.

### What Happens When the Middleware Runs

When a request passes through the authentication middleware, the following occurs:

1. The middleware invokes the **default authentication scheme's handler**.
2. The handler inspects the request for credentials (a cookie, a JWT in the `Authorization` header, etc.).
3. If valid credentials are found, the handler creates a `ClaimsPrincipal` representing the user.
4. The middleware sets `HttpContext.User` to that principal.
5. If no credentials are found or they are invalid, `HttpContext.User` remains an anonymous (unauthenticated) principal.

> [!tip]
> The authentication middleware does **not** reject unauthenticated requests. It simply populates (or does not populate) the user. It is the **authorization** middleware that decides whether to allow or reject the request.

> [!summary] Section Summary
> The authentication middleware reads credentials from the request, validates them through a handler, and sets `HttpContext.User`. It must be placed after `UseRouting()` and before `UseAuthorization()` in the pipeline. It never rejects requests on its own.

---

## Authentication Schemes

> [!info] Definition
> An **authentication scheme** is a named configuration that tells ASP.NET Core *how* to authenticate a request. Each scheme is associated with a specific handler that knows how to read and validate a particular credential type.

Think of schemes as different "strategies" for authentication. A cookie scheme knows how to read encrypted cookies. A JWT Bearer scheme knows how to validate JSON Web Tokens. An OAuth scheme knows how to redirect to an external provider.

### Registering Schemes

Schemes are registered during service configuration in `Program.cs`:

```csharp
builder.Services.AddAuthentication(defaultScheme: "Cookies")
    .AddCookie("Cookies", options =>
    {
        options.LoginPath = "/Account/Login";
        options.AccessDeniedPath = "/Account/AccessDenied";
        options.ExpireTimeSpan = TimeSpan.FromHours(2);
    })
    .AddJwtBearer("Bearer", options =>
    {
        options.Authority = "https://login.example.com";
        options.Audience = "my-api";
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true
        };
    });
```

The string passed to `AddCookie()` or `AddJwtBearer()` is the **scheme name**. You reference this name later when you need to specify which scheme to use.

### Common Built-In Schemes

| Scheme Type | NuGet Package | Common Use Case |
|---|---|---|
| Cookie | Built-in | Browser-based web apps |
| JWT Bearer | `Microsoft.AspNetCore.Authentication.JwtBearer` | APIs, SPAs, mobile apps |
| OAuth 2.0 | `Microsoft.AspNetCore.Authentication.OAuth` | External provider login |
| OpenID Connect | `Microsoft.AspNetCore.Authentication.OpenIdConnect` | Enterprise SSO, Azure AD |
| Google/Facebook/etc. | Individual packages | Social login |

> [!summary] Section Summary
> Authentication schemes are named strategies for reading and validating credentials. Each scheme maps to a handler. Multiple schemes can coexist in the same application, and you designate one as the default.

---

## ClaimsPrincipal

> [!info] Definition
> A **ClaimsPrincipal** represents the authenticated user. It is the top-level object accessible via `HttpContext.User` and contains one or more `ClaimsIdentity` objects.

The `ClaimsPrincipal` is the standard .NET security principal used across all of ASP.NET Core. Every request has one -- even anonymous requests (in which case the principal has no authenticated identity).

### Structure

```
ClaimsPrincipal
  |
  +-- ClaimsIdentity (e.g., from Cookie scheme)
  |     |-- Claim: Name = "john.doe"
  |     |-- Claim: Email = "john@example.com"
  |     |-- Claim: Role = "Admin"
  |
  +-- ClaimsIdentity (e.g., from external provider)
        |-- Claim: Name = "john.doe"
        |-- Claim: ProfilePicture = "https://..."
```

### Accessing the Principal

```csharp
// In a controller
public IActionResult Profile()
{
    var user = HttpContext.User;            // ClaimsPrincipal
    var name = user.Identity?.Name;        // Primary identity name
    var isAuth = user.Identity?.IsAuthenticated ?? false;

    var email = user.FindFirst(ClaimTypes.Email)?.Value;
    var roles = user.FindAll(ClaimTypes.Role).Select(c => c.Value);

    return View(new ProfileViewModel(name, email, roles));
}
```

> [!tip]
> `User.Identity.IsAuthenticated` returns `true` only if the identity has an `AuthenticationType` set. An identity created without specifying `AuthenticationType` is considered anonymous.

> [!summary] Section Summary
> `ClaimsPrincipal` is the root object representing the authenticated user. It holds one or more `ClaimsIdentity` objects, each containing claims. Access it via `HttpContext.User` in controllers and middleware.

---

## Claims

> [!info] Definition
> A **Claim** is a key-value pair that states something about the user. Claims are facts asserted by the authentication authority -- such as "this user's name is john.doe" or "this user has the Admin role."

Claims are the building blocks of identity in ASP.NET Core. Rather than having a rigid `User` object with fixed properties, the claims model is flexible -- any piece of information about a user can be expressed as a claim.

### Standard Claim Types

The `System.Security.Claims.ClaimTypes` class defines well-known claim type URIs:

| Friendly Name | ClaimTypes Constant | URI |
|---|---|---|
| Name | `ClaimTypes.Name` | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` |
| Email | `ClaimTypes.Email` | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |
| Role | `ClaimTypes.Role` | `http://schemas.microsoft.com/ws/2008/06/identity/claims/role` |
| NameIdentifier | `ClaimTypes.NameIdentifier` | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier` |

### Creating Claims

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, "john.doe"),
    new Claim(ClaimTypes.Email, "john@example.com"),
    new Claim(ClaimTypes.Role, "Admin"),
    new Claim(ClaimTypes.Role, "Editor"),           // Multiple roles
    new Claim(ClaimTypes.NameIdentifier, "12345"),  // Unique user ID
    new Claim("Department", "Engineering"),          // Custom claim
    new Claim("EmployeeId", "EMP-0042"),             // Custom claim
    new Claim("HireDate", "2023-01-15")              // Custom claim
};
```

### Reading Claims

```csharp
// Find first matching claim
string? email = User.FindFirst(ClaimTypes.Email)?.Value;

// Find all matching claims (useful for roles)
IEnumerable<string> roles = User.FindAll(ClaimTypes.Role)
    .Select(c => c.Value);

// Check if a claim exists with a specific value
bool isAdmin = User.HasClaim(ClaimTypes.Role, "Admin");

// Check if a claim of a type exists at all
bool hasEmail = User.HasClaim(c => c.Type == ClaimTypes.Email);
```

> [!warning] Common Misconception
> Claims are **not** permissions. A claim states a fact about the user ("this user is in the Admin role"). Authorization policies then *interpret* those claims to make access decisions. The claim itself does not grant or deny anything.

> [!summary] Section Summary
> Claims are key-value pairs that describe the user. Standard claim types exist for common attributes like Name, Email, and Role, but you can define custom claims for any purpose. Claims are read from the `ClaimsPrincipal` using `FindFirst()`, `FindAll()`, and `HasClaim()`.

---

## ClaimsIdentity

> [!info] Definition
> A **ClaimsIdentity** represents a single source of identity information. It contains a collection of claims and an `AuthenticationType` that indicates how the user was authenticated. Think of it as a single ID card.

A `ClaimsPrincipal` can hold multiple `ClaimsIdentity` objects -- just as a person might carry a driver's license, a passport, and a work badge. Each identity comes from a different authentication source and may contain different claims.

### Creating a ClaimsIdentity

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, "john.doe"),
    new Claim(ClaimTypes.Email, "john@example.com"),
    new Claim(ClaimTypes.Role, "Admin")
};

// The second parameter is the AuthenticationType
var identity = new ClaimsIdentity(claims, "CookieAuth");
```

> [!warning] Common Misconception
> If you create a `ClaimsIdentity` without specifying an `AuthenticationType`, the identity's `IsAuthenticated` property will return `false`. This is a common source of bugs -- the user has claims but appears unauthenticated:
> ```csharp
> // BUG: IsAuthenticated will be false!
> var identity = new ClaimsIdentity(claims);
> Console.WriteLine(identity.IsAuthenticated); // false
>
> // CORRECT: Specify AuthenticationType
> var identity = new ClaimsIdentity(claims, "CookieAuth");
> Console.WriteLine(identity.IsAuthenticated); // true
> ```

### Building a Principal from Identities

```csharp
// Identity from local login
var localClaims = new List<Claim>
{
    new Claim(ClaimTypes.Name, "john.doe"),
    new Claim(ClaimTypes.Role, "Admin")
};
var localIdentity = new ClaimsIdentity(localClaims, "LocalAuth");

// Identity from external provider (e.g., Google)
var externalClaims = new List<Claim>
{
    new Claim("picture", "https://lh3.google.com/..."),
    new Claim(ClaimTypes.Email, "john@gmail.com")
};
var externalIdentity = new ClaimsIdentity(externalClaims, "Google");

// Combine into a single principal
var principal = new ClaimsPrincipal(new[] { localIdentity, externalIdentity });
```

### Name and Role Claim Types

By default, `ClaimsIdentity` looks for `ClaimTypes.Name` when you access the `Name` property, and `ClaimTypes.Role` when checking roles. You can customize this:

```csharp
// Use custom claim types for name and role
var identity = new ClaimsIdentity(
    claims: claims,
    authenticationType: "Custom",
    nameType: "preferred_username",   // Instead of ClaimTypes.Name
    roleType: "groups"                // Instead of ClaimTypes.Role
);
```

> [!tip]
> JWT tokens from providers like Azure AD or Auth0 often use non-standard claim names (e.g., `preferred_username` instead of `ClaimTypes.Name`). Use the `nameType` and `roleType` parameters to map them correctly without manually transforming claims.

> [!summary] Section Summary
> A `ClaimsIdentity` groups claims from a single authentication source. It must have an `AuthenticationType` set to be considered authenticated. A `ClaimsPrincipal` can hold multiple identities, and the name/role claim type mappings are customizable per identity.

---

## How Authentication Works at a High Level

Understanding the complete flow from an incoming request to an authenticated user is essential. Here is the step-by-step process:

### The Complete Flow

```
Client Request
     |
     v
[1] Routing Middleware --> determines which endpoint will handle the request
     |
     v
[2] Authentication Middleware
     |  - Selects the default authentication handler
     |  - Handler inspects the request for credentials
     |    (reads cookie, extracts JWT from Authorization header, etc.)
     |  - If valid: creates ClaimsPrincipal, sets HttpContext.User
     |  - If invalid/missing: HttpContext.User remains anonymous
     |
     v
[3] Authorization Middleware
     |  - Checks [Authorize] requirements on the matched endpoint
     |  - If user is authenticated and authorized: continue
     |  - If user is not authenticated: CHALLENGE (401 / redirect to login)
     |  - If user is authenticated but not authorized: FORBID (403)
     |
     v
[4] Endpoint Execution (Controller Action / Minimal API handler)
```

### A Concrete Example: Cookie Authentication

```csharp
// 1. User submits login form
[HttpPost("login")]
public async Task<IActionResult> Login(LoginModel model)
{
    // 2. Validate credentials against your database
    var user = await _userService.ValidateCredentials(model.Email, model.Password);
    if (user is null)
        return View("Login", new { Error = "Invalid credentials" });

    // 3. Create claims describing the user
    var claims = new List<Claim>
    {
        new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
        new Claim(ClaimTypes.Name, user.DisplayName),
        new Claim(ClaimTypes.Email, user.Email),
        new Claim(ClaimTypes.Role, user.Role)
    };

    // 4. Create the identity and principal
    var identity = new ClaimsIdentity(claims, "CookieAuth");
    var principal = new ClaimsPrincipal(identity);

    // 5. Sign in -- this serializes the principal into an encrypted cookie
    await HttpContext.SignInAsync("Cookies", principal, new AuthenticationProperties
    {
        IsPersistent = model.RememberMe,
        ExpiresUtc = DateTimeOffset.UtcNow.AddHours(8)
    });

    return RedirectToAction("Index", "Home");
}

// 6. On subsequent requests, the cookie middleware automatically:
//    - Reads the cookie
//    - Decrypts it
//    - Rebuilds the ClaimsPrincipal
//    - Sets HttpContext.User
```

> [!ad-note]
> The `SignInAsync` method does not validate credentials. It takes an already-validated principal and persists it (usually as a cookie). Credential validation is **your** responsibility in the login action.

> [!summary] Section Summary
> The authentication flow runs on every request: the handler reads credentials, validates them, and builds a `ClaimsPrincipal`. For cookie auth, the initial sign-in serializes the principal into an encrypted cookie, and subsequent requests automatically deserialize it. The authorization middleware then uses the principal to enforce access rules.

---

## Default Scheme vs Challenge Scheme vs Forbid Scheme

ASP.NET Core uses several distinct "scheme selectors" to determine which handler to invoke for different operations. Understanding these is critical for applications with multiple authentication schemes.

### The Four Scheme Types

| Scheme Type | Purpose | When It Fires |
|---|---|---|
| **DefaultScheme** | Fallback for all operations if no specific scheme is set | Any operation without an explicit scheme |
| **DefaultAuthenticateScheme** | Which handler reads credentials from the request | Every request (via authentication middleware) |
| **DefaultChallengeScheme** | What happens when an unauthenticated user hits a protected resource | Authorization middleware detects no authenticated user |
| **DefaultForbidScheme** | What happens when an authenticated user lacks permission | Authorization middleware detects insufficient permissions |

### Behavior of Each

**Authenticate** -- the handler that reads and validates credentials on every request:
```csharp
// The cookie handler reads the auth cookie and builds the principal
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = "Cookies";
});
```

**Challenge** -- invoked when an unauthenticated user accesses a protected endpoint. For cookies, this typically means redirecting to a login page. For JWT, this means returning a `401 Unauthorized` response:
```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultChallengeScheme = "Cookies"; // Redirects to login
    // vs.
    options.DefaultChallengeScheme = "Bearer";  // Returns 401
});
```

**Forbid** -- invoked when an authenticated user lacks permission. For cookies, this redirects to an "Access Denied" page. For JWT, this returns `403 Forbidden`:
```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultForbidScheme = "Cookies"; // Redirects to Access Denied page
});
```

### Simplified Configuration

When you pass a single string to `AddAuthentication()`, it sets **all** default schemes at once:

```csharp
// This sets DefaultScheme, which is used as fallback for
// DefaultAuthenticateScheme, DefaultChallengeScheme,
// DefaultSignInScheme, DefaultSignOutScheme, and DefaultForbidScheme
builder.Services.AddAuthentication("Cookies")
    .AddCookie("Cookies");
```

> [!tip]
> For most applications with a single authentication scheme, just pass the scheme name to `AddAuthentication()`. You only need to set individual scheme types when mixing multiple schemes (e.g., cookies for browsers and JWT for APIs).

> [!summary] Section Summary
> ASP.NET Core distinguishes between Authenticate (read credentials), Challenge (handle unauthenticated access), and Forbid (handle unauthorized access) operations. Each can be mapped to a different handler. The `DefaultScheme` acts as a fallback for all operations when specific defaults are not set.

---

## Multiple Authentication Schemes

Real-world applications often need to support multiple authentication methods -- for example, cookies for browser-based users and JWT tokens for API clients.

### Configuration

```csharp
builder.Services.AddAuthentication(options =>
{
    // Default for browser requests
    options.DefaultScheme = "Cookies";
    options.DefaultChallengeScheme = "Cookies";
})
.AddCookie("Cookies", options =>
{
    options.LoginPath = "/Account/Login";
    options.ExpireTimeSpan = TimeSpan.FromHours(2);
})
.AddJwtBearer("Bearer", options =>
{
    options.Authority = "https://auth.example.com";
    options.Audience = "my-api";
});
```

### Selecting a Scheme Per Endpoint

Use the `[Authorize]` attribute to specify which scheme an endpoint should use:

```csharp
// Uses the default scheme (Cookies)
[Authorize]
public class DashboardController : Controller
{
    public IActionResult Index() => View();
}

// Explicitly uses JWT Bearer
[Authorize(AuthenticationSchemes = "Bearer")]
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok(_products);
}

// Accepts EITHER cookies OR JWT
[Authorize(AuthenticationSchemes = "Cookies,Bearer")]
[Route("api/[controller]")]
public class HybridController : ControllerBase
{
    [HttpGet]
    public IActionResult Get() => Ok("Authenticated via either scheme");
}
```

### Policy-Based Scheme Selection

For more control, you can create authorization policies that specify schemes:

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("ApiPolicy", policy =>
    {
        policy.AuthenticationSchemes.Add("Bearer");
        policy.RequireAuthenticatedUser();
    });

    options.AddPolicy("WebPolicy", policy =>
    {
        policy.AuthenticationSchemes.Add("Cookies");
        policy.RequireAuthenticatedUser();
    });
});
```

```csharp
[Authorize(Policy = "ApiPolicy")]
public class SecureApiController : ControllerBase { }
```

> [!warning] Common Misconception
> When you specify `AuthenticationSchemes` on the `[Authorize]` attribute, **only** those schemes are used to authenticate the request for that endpoint. The default scheme is ignored. If the user authenticated via cookies but the endpoint requires `"Bearer"`, the user will be treated as unauthenticated for that endpoint.

> [!summary] Section Summary
> Multiple authentication schemes let you support different client types (browsers, APIs, mobile apps) simultaneously. Schemes can be selected per-endpoint using the `[Authorize]` attribute or via authorization policies. When schemes are specified explicitly, the default scheme is overridden for that endpoint.

---

## The Authorize Attribute

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

---

## The AllowAnonymous Attribute

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

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Authentication** in ASP.NET Core is a flexible, scheme-based system built around the `ClaimsPrincipal` model.
>
> **Core concepts:**
> - Authentication (who are you?) runs before Authorization (what can you do?)
> - The authentication middleware sits between `UseRouting()` and `UseAuthorization()` in the pipeline
> - **Authentication schemes** are named strategies (Cookie, JWT Bearer, OAuth) registered during service configuration
> - Each scheme has a **handler** that reads credentials from the request and produces a `ClaimsPrincipal`
>
> **The identity model:**
> - A `ClaimsPrincipal` holds one or more `ClaimsIdentity` objects
> - Each `ClaimsIdentity` represents credentials from a single source (like one ID card)
> - `Claims` are key-value pairs describing the user (name, email, roles, custom data)
> - An identity must have an `AuthenticationType` set to be considered authenticated
>
> **Scheme operations:**
> - **Authenticate** -- read and validate credentials on every request
> - **Challenge** -- handle unauthenticated access (redirect to login or return 401)
> - **Forbid** -- handle unauthorized access (redirect to access denied or return 403)
>
> **Access control attributes:**
> - `[Authorize]` requires authentication and optionally enforces roles or policies
> - `[AllowAnonymous]` overrides authorization to permit unauthenticated access
> - A global `FallbackPolicy` can require authentication by default across the entire application
>
> This overview covers the foundational layer. The specific implementations -- [[Cookie Authentication]], [[JWT Authentication]], and [[ASP.NET Core Identity]] -- build on these core concepts to provide complete authentication solutions.

---

## Related Topics

- [[Cookie Authentication]] -- implementing cookie-based authentication for web applications
- [[JWT Authentication]] -- implementing token-based authentication for APIs
- [[ASP.NET Core Identity]] -- the full identity framework with user management, passwords, and roles
- [[Authorization Policies]] -- creating custom authorization requirements beyond simple roles
- [[Middleware Overview]] -- understanding the ASP.NET Core middleware pipeline
- [[Request Pipeline]] -- how requests flow through the application

---

## Further Reading

- [Introduction to authentication in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/) -- Microsoft Docs
- [Introduction to authorization in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/introduction) -- Microsoft Docs
- [Claims-based authorization](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/claims) -- Microsoft Docs
- [Use cookie authentication without ASP.NET Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie) -- Microsoft Docs
- [Overview of ASP.NET Core authentication](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/) -- Microsoft Docs
- [Authorize with a specific scheme](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/limitingidentitybyscheme) -- Microsoft Docs
