---
tags:
  - csharp
  - asp-net-core
  - filters
  - controllers
  - cross-cutting
---


Authorization filters run **first** in the pipeline, before anything else. They decide whether the current user is allowed to proceed.

- **Interface**: `IAuthorizationFilter` / `IAsyncAuthorizationFilter`
- Can **short-circuit** the entire pipeline by setting `context.Result`
- Run before model binding, before resource filters

```csharp
public class RequireClaimFilter : IAuthorizationFilter
{
    private readonly string _claimType;

    public RequireClaimFilter(string claimType)
    {
        _claimType = claimType;
    }

    public void OnAuthorization(AuthorizationFilterContext context)
    {
        bool hasClaim = context.HttpContext.User.HasClaim(c => c.Type == _claimType);

        if (!hasClaim)
        {
            context.Result = new ForbidResult(); // Short-circuits the pipeline
        }
    }
}
```

```ad-warning
You should **rarely** implement custom authorization filters. The built-in policy-based authorization system is far more flexible, testable, and maintainable. Use `[Authorize(Policy = "...")]` instead.

See [[17.09 - Authentication and Authorization]] for the policy-based approach.
```
