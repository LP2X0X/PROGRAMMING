---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


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
