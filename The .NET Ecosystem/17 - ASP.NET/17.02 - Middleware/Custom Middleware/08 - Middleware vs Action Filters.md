---
tags:
  - csharp
  - asp-net-core
  - middleware
  - custom
---

## Middleware vs Action Filters

**Middleware** and **action filters** both intercept request processing, but they operate at different levels of the ASP.NET Core pipeline.

**Middleware** operates on the raw HTTP pipeline. It runs for every request -- including static files, health checks, SignalR, gRPC, and minimal API endpoints. It has no knowledge of MVC concepts like controllers, actions, or model binding.

**Action filters** operate inside the MVC/Razor Pages pipeline. They run only for requests that are routed to a controller action or Razor Page handler. They have access to MVC-specific context like `ActionExecutingContext`, model binding results, and action arguments.

| Aspect | Middleware | Action Filters |
|---|---|---|
| **Pipeline level** | HTTP pipeline (all requests) | MVC pipeline (controller actions only) |
| **Scope** | Every request including static files, gRPC, minimal APIs | Only requests routed to MVC controllers/Razor Pages |
| **Access to MVC context** | No -- only `HttpContext` | Yes -- `ActionExecutingContext`, model binding, action arguments |
| **DI support** | Constructor + `InvokeAsync` params | Constructor injection, `[ServiceFilter]`, `[TypeFilter]` |
| **Ordering** | Pipeline order (first registered = outermost) | Filter order (global, controller, action levels) |
| **Short-circuiting** | Skip `next()` to short-circuit entire pipeline | Set `context.Result` to short-circuit action execution |
| **Best for** | Cross-cutting concerns: logging, CORS, timing, error handling, correlation IDs, compression | MVC-specific concerns: validation, authorization attributes, caching, result transformation |

### Decision Guide

Use **middleware** when:
- The logic applies to all requests regardless of endpoint type
- You need to run before routing or authentication
- You are dealing with raw HTTP concerns (headers, status codes, request body)
- You want to affect non-MVC endpoints (minimal APIs, gRPC, static files)

Use **action filters** when:
- The logic is specific to MVC controller actions
- You need access to action arguments, model binding, or `ActionResult`
- You want to apply the filter selectively via attributes (`[ServiceFilter]`, `[TypeFilter]`)
- The behavior is closely tied to business logic rather than infrastructure

> [!example]
> **Request timing**: Use middleware -- you want to time all requests, not just MVC actions.
> **Input validation logging**: Use an action filter -- you need access to the model binding result.
> **API key check**: Can be either, but middleware is better if you also serve minimal API endpoints.
> **Audit logging of action parameters**: Use an action filter -- you need `ActionExecutingContext.ActionArguments`.

> [!warning] Common Misconception
> Developers sometimes put cross-cutting concerns like exception handling into an `IExceptionFilter` and assume it catches all exceptions. It does not -- `IExceptionFilter` only catches exceptions thrown during action execution and result execution within the MVC pipeline. Exceptions thrown in middleware (before the MVC pipeline) or in non-MVC endpoints will bypass it entirely. Use exception handling middleware for truly global coverage.

> [!summary] Section Summary
> Middleware runs for all HTTP requests at the pipeline level; action filters run only for MVC controller actions inside the MVC pipeline. Use middleware for infrastructure cross-cutting concerns (logging, timing, error handling) and action filters for MVC-specific concerns (validation, authorization attributes, result transformation). Exception handling should be middleware for full coverage.
