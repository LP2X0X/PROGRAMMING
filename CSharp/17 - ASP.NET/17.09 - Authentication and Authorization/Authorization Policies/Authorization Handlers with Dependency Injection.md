---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


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
