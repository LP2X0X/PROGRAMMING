---
tags:
  - csharp
  - asp-net-core
  - authentication
  - security
---


> [!tip] Complete Summary
> **Authentication** in ASP.NET Core is a flexible, scheme-based system built around the `ClaimsPrincipal` model.
>
> **Core concepts:**
> - Authentication (who are you?) runs before Authorization (what can you do?)
> - The authentication middleware sits between `UseRouting()` and `UseAuthorization()` in the pipeline
> - **Authentication schemes** are named strategies (Cookie, JWT Bearer, OAuth) registered during service configuration
> - Each scheme has a **handler** that reads credentials from the request and produces a `ClaimsPrincipal`
>
> **The identity model:**
> - A `ClaimsPrincipal` holds one or more `ClaimsIdentity` objects
> - Each `ClaimsIdentity` represents credentials from a single source (like one ID card)
> - `Claims` are key-value pairs describing the user (name, email, roles, custom data)
> - An identity must have an `AuthenticationType` set to be considered authenticated
>
> **Scheme operations:**
> - **Authenticate** -- read and validate credentials on every request
> - **Challenge** -- handle unauthenticated access (redirect to login or return 401)
> - **Forbid** -- handle unauthorized access (redirect to access denied or return 403)
>
> **Access control attributes:**
> - `[Authorize]` requires authentication and optionally enforces roles or policies
> - `[AllowAnonymous]` overrides authorization to permit unauthenticated access
> - A global `FallbackPolicy` can require authentication by default across the entire application
>
> This overview covers the foundational layer. The specific implementations -- [[Cookie Authentication]], [[JWT Authentication]], and [[ASP.NET Core Identity]] -- build on these core concepts to provide complete authentication solutions.
