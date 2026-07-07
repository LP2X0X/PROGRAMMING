---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


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
