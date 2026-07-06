---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


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
