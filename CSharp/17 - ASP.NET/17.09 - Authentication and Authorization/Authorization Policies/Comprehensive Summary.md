---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


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
