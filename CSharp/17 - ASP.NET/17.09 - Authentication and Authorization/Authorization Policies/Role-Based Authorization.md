---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


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
