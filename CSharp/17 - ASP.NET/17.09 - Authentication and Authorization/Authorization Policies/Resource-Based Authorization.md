---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


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
