---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


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
