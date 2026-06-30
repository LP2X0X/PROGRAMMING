---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
  - request
---

## Comprehensive Summary

> [!tip] Complete Summary
> The ASP.NET Core request pipeline is a sequential chain of middleware components that process every HTTP request. The standard ordering is: (1) Exception Handling, (2) HSTS, (3) HTTPS Redirection, (4) Static Files, (5) Routing, (6) CORS, (7) Authentication, (8) Authorization, (9) Custom Middleware, (10) Endpoints.
>
> Each position exists for a specific reason. Exception handling wraps everything to catch errors from any component. HSTS and HTTPS redirection enforce security before any content is served. Static files short-circuit the pipeline to avoid unnecessary routing and auth overhead. Routing selects the endpoint so that CORS, authentication, and authorization can access endpoint metadata. Authorization depends on both routing (to know which endpoint) and authentication (to know who the user is). Custom middleware runs in a fully resolved context with access to user identity and endpoint information. Endpoints execute last.
>
> The **endpoint routing split** is the critical architectural concept: `UseRouting()` selects the endpoint, and middleware between routing and endpoint execution can read endpoint metadata. This is how ASP.NET Core applies endpoint-specific authorization policies.
>
> **Short-circuiting** allows middleware to return a response without passing the request further. Static files short-circuit for performance; custom middleware can short-circuit for maintenance mode, rate limiting, or validation failures.
>
> **Pipeline branching** via `Map()`, `MapWhen()`, and `UseWhen()` creates separate middleware chains for different request types. **Terminal middleware** via `app.Run()` always ends the pipeline and is used as a catch-all fallback.

## Related Topics

- [[Middleware Overview]]
- [[Custom Middleware]]
- [[Built-in Middleware]]
- [[Authentication and Authorization]]
- [[Routing in ASP.NET Core]]
- [[Static Files Configuration]]
- [[CORS Configuration]]
- [[Error Handling Middleware]]
- [[Endpoint Routing]]
- [[Minimal APIs]]

## Further Reading

- [[Dependency Injection in ASP.NET Core]]
- [[Program.cs and Host Builder]]
- [[Kestrel Web Server]]
- [[Reverse Proxy Configuration]]
- [[Security Headers]]
- [[Rate Limiting Middleware]]
- [[Health Checks]]
- [[SignalR Hubs]]
