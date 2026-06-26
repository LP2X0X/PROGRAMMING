---
tags: [csharp, asp-net-core, authorization, policies, security]
aliases: [Auth Policies, Policy-Based Authorization, ASP.NET Core Authorization]
status: complete
date: 2026-06-18
---

# Authorization Policies

## Table of Contents

- [[#What Is Authorization]]
- [[#Simple Authorization]]
- [[#Role-Based Authorization]]
- [[#Claims-Based Authorization]]
- [[#Policy-Based Authorization]]
- [[#Resource-Based Authorization]]
- [[#Authorize and AllowAnonymous Interaction]]
- [[#Authorization in Razor Views]]
- [[#Authorization in Minimal APIs]]
- [[#Authorization Handlers with Dependency Injection]]
- [[#Complete Real-World Example]]
- [[#Authorization Flow]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]
- [[#Further Reading]]

---

## What Is Authorization

> [!info] Definition
> **Authorization** is the process of determining whether an authenticated user has permission to access a specific resource or perform a specific action. It answers the question: "You are who you say you are -- but are you *allowed* to do this?"

Authorization is distinct from authentication. Authentication establishes *identity* -- who the user is. Authorization establishes *access* -- what the user can do. These two concepts work together but serve fundamentally different purposes.

| Concern          | Question Answered      | Example                          |
| ---------------- | ---------------------- | -------------------------------- |
| Authentication   | Who are you?           | "I am user `john@example.com`"   |
| Authorization    | Are you allowed?       | "Can John access the admin panel?" |

In ASP.NET Core, authorization is handled by middleware that runs *after* authentication middleware. The authentication middleware establishes the user's identity (as a `ClaimsPrincipal`), and the authorization middleware evaluates policies against that identity to decide whether access is granted.

See [[Authentication Overview]] for details on how identity is established before authorization takes place.

> [!summary] Section Summary
> Authorization decides what an authenticated user can do. It runs after authentication in the middleware pipeline and evaluates policies against the user's claims and roles.

---

## Simple Authorization

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

---

## Role-Based Authorization

Role-based authorization restricts access based on the roles assigned to a user. Roles are a familiar concept from traditional security models -- you assign users to groups like "Admin," "Manager," or "User," and gate access based on group membership.

### Basic Role Check

```csharp
[Authorize(Roles = "Admin")]
public class AdminController : Controller
{
    public IActionResult Dashboard()
    {
        return View();
    }
}
```

### Checking Roles Programmatically

Inside a controller or Razor Page, you can check roles using the `User` property:

```csharp
public IActionResult ManageUsers()
{
    if (User.IsInRole("Admin"))
    {
        // Show admin-level user management
        return View("AdminManageUsers");
    }

    // Show basic user management
    return View("BasicManageUsers");
}
```

### Multiple Roles -- OR Logic vs AND Logic

When you specify multiple roles in a single `[Authorize]` attribute separated by commas, the user must be in **at least one** of those roles (OR logic):

```csharp
// User must be Admin OR Manager (either one is sufficient)
[Authorize(Roles = "Admin,Manager")]
public class ReportController : Controller
{
    public IActionResult ViewReports() => View();
}
```

When you stack multiple `[Authorize]` attributes, the user must satisfy **all** of them (AND logic):

```csharp
// User must be Admin AND Manager (must be in BOTH roles)
[Authorize(Roles = "Admin")]
[Authorize(Roles = "Manager")]
public class SensitiveOperationsController : Controller
{
    public IActionResult PerformSensitiveAction() => View();
}
```

> [!tip]
> Remember: comma-separated roles within one attribute = **OR**. Multiple attributes = **AND**. This is a frequently tested concept and a common source of bugs if misunderstood.

### Limitations of Role-Based Authorization

Role-based authorization works well for simple scenarios, but it has significant limitations:

- **Inflexible** -- roles are coarse-grained. You cannot express fine-grained rules like "users who joined before 2020" or "users in the same department as the resource owner."
- **Doesn't scale** -- as requirements grow, you end up creating dozens of roles to cover every combination of permissions, leading to role explosion.
- **Hard to manage** -- role assignments live in the identity store (database, Active Directory, etc.), making it difficult to change authorization logic without modifying user records.
- **Static** -- roles don't account for dynamic context like time of day, resource ownership, or environmental conditions.

> [!danger]
> Relying solely on roles for complex authorization scenarios leads to unmaintainable code. Prefer claims-based or policy-based authorization for anything beyond simple access control.

> [!summary] Section Summary
> Role-based authorization gates access by user group membership. Comma-separated roles in one attribute use OR logic; stacked attributes use AND logic. While easy to understand, roles become unwieldy for complex requirements -- prefer claims or policies when rules get sophisticated.

---

## Claims-Based Authorization

Claims-based authorization is a step up from roles. Instead of checking broad group memberships, you check specific *claims* -- pieces of information about the user that are carried in their identity token.

> [!info] Definition
> A **claim** is a name-value pair that describes a property of a user. Examples: `email: john@example.com`, `department: IT`, `subscription_level: premium`. Claims are more granular than roles and can represent virtually any user attribute.

### Registering Claim-Based Policies

You register policies in `Program.cs` (or `Startup.cs`) by defining what claims are required:

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("RequireITDepartment", policy =>
        policy.RequireClaim("Department", "IT"));

    options.AddPolicy("RequireEmailVerified", policy =>
        policy.RequireClaim("email_verified", "true"));

    options.AddPolicy("RequirePremiumSubscription", policy =>
        policy.RequireClaim("subscription_level", "premium", "enterprise"));
});
```

> [!ad-note]
> When `RequireClaim` is called with multiple allowed values (like `"premium", "enterprise"` above), the user must have the claim with **any one** of those values.

### Using Claim-Based Policies

Apply the policy using the `[Authorize]` attribute's `Policy` parameter:

```csharp
[Authorize(Policy = "RequireITDepartment")]
public class ITToolsController : Controller
{
    public IActionResult NetworkMonitor() => View();
}
```

### Built-In Policy Requirements

ASP.NET Core provides several built-in methods for constructing policies without writing custom handlers:

```csharp
builder.Services.AddAuthorization(options =>
{
    // Require a specific claim with a specific value
    options.AddPolicy("ITOnly", policy =>
        policy.RequireClaim("Department", "IT"));

    // Require a claim to exist (any value)
    options.AddPolicy("HasDepartment", policy =>
        policy.RequireClaim("Department"));

    // Require a role (equivalent to [Authorize(Roles = "Admin")])
    options.AddPolicy("AdminPolicy", policy =>
        policy.RequireRole("Admin"));

    // Require a specific username
    options.AddPolicy("SpecificUser", policy =>
        policy.RequireUserName("admin@example.com"));

    // Require an assertion (inline logic)
    options.AddPolicy("BusinessHoursOnly", policy =>
        policy.RequireAssertion(context =>
        {
            var hour = DateTime.Now.Hour;
            return hour >= 9 && hour < 17;
        }));
});
```

> [!example] Combining Multiple Requirements
> You can chain requirements within a single policy. **All** requirements must be satisfied for the policy to succeed:
> ```csharp
> options.AddPolicy("SeniorITAdmin", policy =>
> {
>     policy.RequireRole("Admin");
>     policy.RequireClaim("Department", "IT");
>     policy.RequireClaim("experience_years");
> });
> ```
> The user must be an Admin, belong to the IT department, **and** have an `experience_years` claim (any value).

> [!summary] Section Summary
> Claims-based authorization checks specific user attributes rather than broad role memberships. ASP.NET Core provides built-in methods like `RequireClaim`, `RequireRole`, `RequireUserName`, and `RequireAssertion` for common scenarios. Claims offer much finer granularity than roles.

---

## Policy-Based Authorization

Policy-based authorization is the **recommended approach** in ASP.NET Core. It provides maximum flexibility by separating *what is required* (requirements) from *how to evaluate* those requirements (handlers). This clean separation follows the Single Responsibility Principle and makes authorization logic testable and maintainable.

> [!info] Definition
> A **policy** is a collection of one or more **requirements**. Each requirement is evaluated by one or more **handlers**. If all requirements in a policy are satisfied, the policy succeeds and access is granted.

### The Building Blocks

| Component | Interface/Base Class | Purpose |
| --- | --- | --- |
| Requirement | `IAuthorizationRequirement` | Declares *what* is needed (a marker with data) |
| Handler | `AuthorizationHandler<TRequirement>` | Contains the *logic* that evaluates a requirement |
| Policy | Registered via `AddPolicy()` | Groups requirements together under a name |

### Example: MinimumAgeRequirement

**Step 1 -- Define the requirement:**

```csharp
public class MinimumAgeRequirement : IAuthorizationRequirement
{
    public int MinimumAge { get; }

    public MinimumAgeRequirement(int minimumAge)
    {
        MinimumAge = minimumAge;
    }
}
```

`IAuthorizationRequirement` is a marker interface -- it has no members. The requirement class simply holds the data needed for the evaluation (in this case, the minimum age).

**Step 2 -- Implement the handler:**

```csharp
public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        var dateOfBirthClaim = context.User.FindFirst(
            c => c.Type == "DateOfBirth");

        if (dateOfBirthClaim is null)
        {
            // No date of birth claim -- cannot determine age.
            // Do NOT call context.Fail() here. Simply return without
            // calling Succeed, allowing other handlers a chance.
            return Task.CompletedTask;
        }

        var dateOfBirth = DateTime.Parse(dateOfBirthClaim.Value);
        var age = DateTime.Today.Year - dateOfBirth.Year;

        // Adjust if the birthday hasn't occurred yet this year
        if (dateOfBirth.Date > DateTime.Today.AddYears(-age))
            age--;

        if (age >= requirement.MinimumAge)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

**Step 3 -- Register the policy and handler:**

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AtLeast21", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(21)));

    options.AddPolicy("AtLeast18", policy =>
        policy.Requirements.Add(new MinimumAgeRequirement(18)));
});

builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();
```

**Step 4 -- Apply the policy:**

```csharp
[Authorize(Policy = "AtLeast21")]
public class AlcoholPurchaseController : Controller
{
    public IActionResult Purchase() => View();
}
```

> [!tip]
> Notice how the same handler (`MinimumAgeHandler`) serves both the "AtLeast21" and "AtLeast18" policies. The handler logic is parameterized by the requirement data. This reusability is a key advantage of the policy-based model.

### Example: SameDepartmentRequirement

Here is a second example demonstrating a requirement that checks whether the user belongs to the same department as a target value:

**Requirement:**

```csharp
public class SameDepartmentRequirement : IAuthorizationRequirement
{
    public string Department { get; }

    public SameDepartmentRequirement(string department)
    {
        Department = department;
    }
}
```

**Handler:**

```csharp
public class SameDepartmentHandler : AuthorizationHandler<SameDepartmentRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        SameDepartmentRequirement requirement)
    {
        var departmentClaim = context.User.FindFirst("Department");

        if (departmentClaim is not null &&
            departmentClaim.Value == requirement.Department)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

**Registration:**

```csharp
options.AddPolicy("EngineeringOnly", policy =>
    policy.Requirements.Add(new SameDepartmentRequirement("Engineering")));

builder.Services.AddSingleton<IAuthorizationHandler, SameDepartmentHandler>();
```

### Multiple Handlers for One Requirement

A single requirement can have **multiple handlers**. The evaluation rules are:

- If **any** handler calls `context.Succeed(requirement)`, the requirement is considered satisfied (unless another handler explicitly fails it).
- If a handler calls `context.Fail()`, the requirement is **explicitly denied**, and this overrides any successes from other handlers.
- If no handler calls either `Succeed` or `Fail`, the requirement is not satisfied (implicit denial).

> [!warning] Common Misconception
> Not calling `context.Succeed()` is **not** the same as calling `context.Fail()`. Omitting `Succeed` means "I can't confirm this" -- another handler might still succeed. Calling `Fail()` means "I explicitly deny this" -- no other handler can override it.

```csharp
// Handler 1: Succeeds if user has "VIP" claim
public class VipAccessHandler : AuthorizationHandler<PremiumContentRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        PremiumContentRequirement requirement)
    {
        if (context.User.HasClaim(c => c.Type == "VIP"))
            context.Succeed(requirement);

        return Task.CompletedTask;
    }
}

// Handler 2: Succeeds if user has an active subscription
public class SubscriptionAccessHandler : AuthorizationHandler<PremiumContentRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        PremiumContentRequirement requirement)
    {
        var subscriptionClaim = context.User.FindFirst("SubscriptionExpiry");
        if (subscriptionClaim is not null &&
            DateTime.Parse(subscriptionClaim.Value) > DateTime.UtcNow)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

In this example, a user can access premium content if they have a VIP claim **or** an active subscription. Either handler succeeding is sufficient.

> [!summary] Section Summary
> Policy-based authorization separates requirements (what is needed) from handlers (how to evaluate). This is the recommended approach because it is flexible, testable, and reusable. Requirements implement `IAuthorizationRequirement`; handlers extend `AuthorizationHandler<T>`. Multiple handlers can serve one requirement -- any `Succeed` grants access, but a `Fail` overrides all successes.

---

## Resource-Based Authorization

Sometimes authorization depends not only on the user but also on the **resource** being accessed. For example, only the author of a blog post should be able to edit it. This requires *imperative authorization* -- you cannot express it purely with attributes because the resource is not available until runtime.

> [!info] Definition
> **Resource-based authorization** evaluates a user's permissions against a specific resource instance. The handler receives both the user's claims and the resource object, enabling decisions like "is this user the owner of this document?"

### Defining the Requirement and Handler

```csharp
public class SameAuthorRequirement : IAuthorizationRequirement { }

public class DocumentAuthorizationHandler :
    AuthorizationHandler<SameAuthorRequirement, Document>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        SameAuthorRequirement requirement,
        Document resource)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;

        if (userId is not null && userId == resource.AuthorId)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

Note the difference: `AuthorizationHandler<TRequirement, TResource>` accepts a second generic parameter for the resource type.

### Registering the Handler and Policy

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("EditDocument", policy =>
        policy.Requirements.Add(new SameAuthorRequirement()));
});

builder.Services.AddScoped<IAuthorizationHandler, DocumentAuthorizationHandler>();
```

### Using Resource-Based Authorization in a Controller

Since the resource must be loaded before authorization can be evaluated, you use `IAuthorizationService` imperatively:

```csharp
public class DocumentController : Controller
{
    private readonly IAuthorizationService _authorizationService;
    private readonly IDocumentRepository _documentRepository;

    public DocumentController(
        IAuthorizationService authorizationService,
        IDocumentRepository documentRepository)
    {
        _authorizationService = authorizationService;
        _documentRepository = documentRepository;
    }

    public async Task<IActionResult> Edit(int id)
    {
        var document = await _documentRepository.GetByIdAsync(id);

        if (document is null)
            return NotFound();

        var authResult = await _authorizationService.AuthorizeAsync(
            User, document, "EditDocument");

        if (!authResult.Succeeded)
            return Forbid();

        return View(document);
    }
}
```

> [!danger]
> Never skip the authorization check when a resource is involved. Loading a resource and returning it without checking ownership is a classic **Insecure Direct Object Reference (IDOR)** vulnerability. Always call `AuthorizeAsync` before allowing access to sensitive resources.

> [!summary] Section Summary
> Resource-based authorization passes a specific resource instance to the handler alongside the user context. Use `AuthorizationHandler<TRequirement, TResource>` for the handler and `IAuthorizationService.AuthorizeAsync(user, resource, policyName)` for imperative evaluation in controllers.

---

## Authorize and AllowAnonymous Interaction

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

---

## Authorization in Razor Views

Sometimes you need to conditionally show or hide UI elements based on authorization policies. You can inject `IAuthorizationService` into Razor views:

```csharp
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService

<nav>
    <a asp-action="Index" asp-controller="Home">Home</a>
    <a asp-action="Profile" asp-controller="Account">Profile</a>

    @if ((await AuthorizationService.AuthorizeAsync(User, "AdminPolicy")).Succeeded)
    {
        <a asp-action="Dashboard" asp-controller="Admin">Admin Panel</a>
    }

    @if ((await AuthorizationService.AuthorizeAsync(User, "ManagerPolicy")).Succeeded)
    {
        <a asp-action="Reports" asp-controller="Reports">Reports</a>
    }
</nav>
```

> [!warning] Common Misconception
> Hiding UI elements is **not** a security measure. It is a UX convenience. Users can still craft HTTP requests to hit endpoints directly. Always enforce authorization on the server side (controllers, handlers, middleware) in addition to hiding UI elements.

You can also check roles directly in Razor views without injecting `IAuthorizationService`:

```csharp
@if (User.IsInRole("Admin"))
{
    <a asp-action="ManageUsers" asp-controller="Admin">Manage Users</a>
}
```

> [!summary] Section Summary
> Inject `IAuthorizationService` into Razor views to conditionally render UI based on policies. Always enforce authorization on the server side -- hiding UI elements alone is not security.

---

## Authorization in Minimal APIs

Minimal APIs use a fluent API for authorization instead of attributes:

```csharp
// Require a specific policy
app.MapGet("/admin/dashboard", () => Results.Ok("Admin Dashboard"))
    .RequireAuthorization("AdminPolicy");

// Require authentication (equivalent to bare [Authorize])
app.MapGet("/profile", (ClaimsPrincipal user) =>
    Results.Ok($"Hello, {user.Identity?.Name}"))
    .RequireAuthorization();

// Allow anonymous access
app.MapGet("/public", () => Results.Ok("Public endpoint"))
    .AllowAnonymous();

// Multiple policies (all must be satisfied)
app.MapGet("/sensitive", () => Results.Ok("Sensitive data"))
    .RequireAuthorization("AdminPolicy", "TwoFactorEnabled");
```

### Grouping Authorization for Minimal APIs

You can apply authorization to a group of endpoints:

```csharp
var adminGroup = app.MapGroup("/admin")
    .RequireAuthorization("AdminPolicy");

adminGroup.MapGet("/users", () => Results.Ok("User list"));
adminGroup.MapGet("/settings", () => Results.Ok("Settings"));
adminGroup.MapPost("/users", (CreateUserDto dto) => Results.Ok("Created"));
```

All endpoints under `/admin` now require the `AdminPolicy` without repeating the authorization call on each one.

> [!summary] Section Summary
> Minimal APIs use `.RequireAuthorization()` and `.AllowAnonymous()` as fluent extensions. Use `MapGroup` to apply authorization to multiple endpoints at once.

---

## Authorization Handlers with Dependency Injection

Authorization handlers are registered in the DI container, which means they can inject any service -- database contexts, loggers, HTTP clients, configuration, and more.

```csharp
public class OrganizationHandler : AuthorizationHandler<SameOrganizationRequirement>
{
    private readonly ApplicationDbContext _db;
    private readonly ILogger<OrganizationHandler> _logger;

    public OrganizationHandler(
        ApplicationDbContext db,
        ILogger<OrganizationHandler> logger)
    {
        _db = db;
        _logger = logger;
    }

    protected override async Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        SameOrganizationRequirement requirement)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;

        if (userId is null)
        {
            _logger.LogWarning("No user ID claim found during organization check");
            return;
        }

        var user = await _db.Users
            .Include(u => u.Organization)
            .FirstOrDefaultAsync(u => u.Id == userId);

        if (user?.Organization?.Id == requirement.OrganizationId)
        {
            _logger.LogInformation(
                "User {UserId} authorized for organization {OrgId}",
                userId, requirement.OrganizationId);
            context.Succeed(requirement);
        }
    }
}
```

### DI Lifetime Considerations

> [!danger]
> Pay careful attention to service lifetimes when injecting services into handlers. If your handler depends on a **Scoped** service (like `DbContext`), the handler itself **must also be registered as Scoped**. Registering a handler as `Singleton` while it depends on a Scoped service will cause a runtime exception (`InvalidOperationException: Cannot consume scoped service from singleton`).

```csharp
// CORRECT: Handler is Scoped because it depends on Scoped DbContext
builder.Services.AddScoped<IAuthorizationHandler, OrganizationHandler>();

// WRONG: Singleton handler consuming Scoped DbContext -- will throw at runtime
// builder.Services.AddSingleton<IAuthorizationHandler, OrganizationHandler>();
```

| Handler Dependency Lifetime | Handler Registration |
| --- | --- |
| No dependencies / Singleton only | `AddSingleton` |
| Scoped services (DbContext, etc.) | `AddScoped` |
| Transient services | `AddScoped` or `AddTransient` |

> [!tip]
> When in doubt, register your handler as **Scoped**. It is safe for all dependency lifetimes and aligns with the per-request nature of web applications.

> [!summary] Section Summary
> Handlers participate fully in DI and can inject any registered service. Match the handler's DI lifetime to its most restrictive dependency -- use Scoped when the handler depends on a Scoped service like `DbContext`.

---

## Complete Real-World Example

This section demonstrates a multi-tenant application where users can only access data belonging to their own organization. It combines custom requirements, handlers, attribute-based authorization, and imperative authorization.

### Requirements

```csharp
// Requirement: user must belong to the specified organization
public class OrganizationMemberRequirement : IAuthorizationRequirement { }

// Requirement: user must be an admin of their organization
public class OrganizationAdminRequirement : IAuthorizationRequirement { }
```

### Handlers

```csharp
public class OrganizationMemberHandler
    : AuthorizationHandler<OrganizationMemberRequirement, OrganizationResource>
{
    private readonly ApplicationDbContext _db;

    public OrganizationMemberHandler(ApplicationDbContext db) => _db = db;

    protected override async Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        OrganizationMemberRequirement requirement,
        OrganizationResource resource)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        if (userId is null) return;

        var isMember = await _db.OrganizationMembers
            .AnyAsync(m => m.UserId == userId
                        && m.OrganizationId == resource.OrganizationId);

        if (isMember)
            context.Succeed(requirement);
    }
}

public class OrganizationAdminHandler
    : AuthorizationHandler<OrganizationAdminRequirement>
{
    private readonly ApplicationDbContext _db;

    public OrganizationAdminHandler(ApplicationDbContext db) => _db = db;

    protected override async Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        OrganizationAdminRequirement requirement)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        if (userId is null) return;

        var orgIdClaim = context.User.FindFirst("OrganizationId");
        if (orgIdClaim is null) return;

        var isAdmin = await _db.OrganizationMembers
            .AnyAsync(m => m.UserId == userId
                        && m.OrganizationId == orgIdClaim.Value
                        && m.Role == "Admin");

        if (isAdmin)
            context.Succeed(requirement);
    }
}
```

### Supporting Types

```csharp
public class OrganizationResource
{
    public string OrganizationId { get; set; } = default!;
}

public class OrganizationMember
{
    public string UserId { get; set; } = default!;
    public string OrganizationId { get; set; } = default!;
    public string Role { get; set; } = default!;
}
```

### Program.cs Registration

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie();

// Register authorization policies
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("SameOrganization", policy =>
        policy.Requirements.Add(new OrganizationMemberRequirement()));

    options.AddPolicy("OrganizationAdmin", policy =>
    {
        policy.Requirements.Add(new OrganizationMemberRequirement());
        policy.Requirements.Add(new OrganizationAdminRequirement());
    });

    // Fallback: require authentication by default
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// Register handlers as Scoped (they depend on Scoped DbContext)
builder.Services.AddScoped<IAuthorizationHandler, OrganizationMemberHandler>();
builder.Services.AddScoped<IAuthorizationHandler, OrganizationAdminHandler>();

builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

### Controller Using Both Attribute and Imperative Authorization

```csharp
[Authorize(Policy = "OrganizationAdmin")]
public class OrganizationSettingsController : Controller
{
    private readonly IAuthorizationService _authService;
    private readonly ApplicationDbContext _db;

    public OrganizationSettingsController(
        IAuthorizationService authService,
        ApplicationDbContext db)
    {
        _authService = authService;
        _db = db;
    }

    // Attribute-based: requires OrganizationAdmin policy
    public IActionResult Index()
    {
        return View();
    }

    // Imperative: checks resource-level organization membership
    public async Task<IActionResult> ViewProject(string projectId)
    {
        var project = await _db.Projects
            .Include(p => p.Organization)
            .FirstOrDefaultAsync(p => p.Id == projectId);

        if (project is null)
            return NotFound();

        var resource = new OrganizationResource
        {
            OrganizationId = project.Organization.Id
        };

        var authResult = await _authService.AuthorizeAsync(
            User, resource, "SameOrganization");

        if (!authResult.Succeeded)
            return Forbid();

        return View(project);
    }
}
```

> [!example] What Happens at Runtime
> 1. A request arrives for `ViewProject("proj-123")`.
> 2. The `[Authorize(Policy = "OrganizationAdmin")]` attribute is evaluated first -- the user must be an organization admin.
> 3. If that passes, the action method runs. It loads the project, then imperatively checks `SameOrganization` -- the user must also be a member of the project's specific organization.
> 4. Both checks must pass for the user to see the project.

> [!summary] Section Summary
> Real-world authorization often combines attribute-based policies on controllers with imperative checks inside actions. This multi-tenant example demonstrates how custom requirements and handlers work together with DI-injected database access to enforce organization-level boundaries.

---

## Authorization Flow

The following diagram shows how an HTTP request moves through the authorization pipeline:

```
Request
  |
  v
+---------------------------+
| Authentication Middleware |  --> Establishes ClaimsPrincipal (who is the user?)
+---------------------------+
  |
  v
+---------------------------+
| Authorization Middleware  |  --> Evaluates policies (is the user allowed?)
+---------------------------+
  |
  v
+---------------------------+
| Policy Evaluation         |  --> Finds the policy for the endpoint
| (which requirements?)     |
+---------------------------+
  |
  v
+---------------------------+
| Requirement Handlers      |  --> Each handler evaluates its requirement
| - Handler A: Succeed()    |
| - Handler B: (no action)  |
| - Handler C: Succeed()    |
+---------------------------+
  |
  v
+-------------------+     +-------------------+
| All requirements  | YES | Access Granted    |
| satisfied?        |---->| (200 OK)          |
+-------------------+     +-------------------+
  | NO
  v
+-------------------+     +-------------------+
| User              | YES | 401 Unauthorized  |
| authenticated?    | NO  | (redirect/reject) |
+-------------------+     +-------------------+
  | YES
  v
+-------------------+
| 403 Forbidden     |
| (access denied)   |
+-------------------+
```

> [!ad-note]
> The distinction between `401` and `403` is important:
> - **401 Unauthorized** -- the user is not authenticated. The response challenges the user to provide credentials.
> - **403 Forbidden** -- the user is authenticated but does not have permission. No amount of re-authenticating will help.

> [!summary] Section Summary
> Authorization flows through authentication middleware (identity), authorization middleware (policy lookup), and requirement handlers (evaluation). Unauthenticated users receive 401; authenticated but unauthorized users receive 403.

---

## Comprehensive Summary

> [!tip] Complete Summary
> ASP.NET Core provides a layered authorization system that scales from simple to complex scenarios:
>
> **Simple authorization** -- the bare `[Authorize]` attribute -- requires only that the user is authenticated.
>
> **Role-based authorization** gates access by role membership. Comma-separated roles in one attribute use OR logic; stacked attributes use AND logic. Roles are easy to understand but don't scale well for complex requirements.
>
> **Claims-based authorization** checks specific user attributes (claims) and is more flexible than roles. Built-in helpers like `RequireClaim`, `RequireRole`, `RequireUserName`, and `RequireAssertion` cover common scenarios without custom code.
>
> **Policy-based authorization** is the recommended approach. It separates requirements (`IAuthorizationRequirement`) from evaluation logic (`AuthorizationHandler<T>`), making authorization testable, reusable, and composable. Multiple handlers can evaluate one requirement -- any `Succeed` grants access, but a `Fail` overrides all successes.
>
> **Resource-based authorization** extends policies by passing a resource instance to the handler, enabling ownership checks and per-resource access control. Use `IAuthorizationService.AuthorizeAsync` for imperative evaluation when the resource is only available at runtime.
>
> **Key patterns to remember:**
> - `[AllowAnonymous]` always overrides `[Authorize]`
> - Set a `FallbackPolicy` for secure-by-default behavior
> - Match handler DI lifetimes to their dependencies (Scoped handler for Scoped DbContext)
> - Hiding UI elements is not security -- always enforce on the server
> - 401 = not authenticated; 403 = authenticated but not authorized
>
> Authorization policies compose cleanly: a single policy can require multiple requirements, each evaluated by dedicated handlers that can query databases, call external services, or perform any logic needed.

---

## Related Topics

- [[Authentication Overview]] -- understanding identity before authorization
- [[Cookie Authentication]] -- cookie-based authentication schemes
- [[JWT Authentication]] -- token-based authentication for APIs
- [[ASP.NET Core Identity]] -- the built-in user management system

---

## Further Reading

- [Microsoft Docs: Introduction to Authorization in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/introduction)
- [Microsoft Docs: Policy-Based Authorization](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies)
- [Microsoft Docs: Resource-Based Authorization](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased)
- [Microsoft Docs: Role-Based Authorization](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/roles)
- [Microsoft Docs: Claims-Based Authorization](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/claims)
- [Andrew Lock: ASP.NET Core in Action (Security Chapters)](https://andrewlock.net)
