---
tags:
  - csharp
  - asp-net-core
  - routing
  - endpoints
---


## Comprehensive Summary

> [!tip] Complete Summary
> **Endpoint routing** is the unified routing system in modern ASP.NET Core (since .NET Core 3.0) that solves the fundamental problem of the old MVC-internal routing: middleware could not see which endpoint would handle a request. The **two-phase model** -- Phase 1 (`UseRouting()`) selects the endpoint, Phase 2 executes it -- creates a window where middleware like authorization, CORS, rate limiting, and output caching can inspect the selected endpoint's metadata before it runs.
>
> Endpoints are registered via `Map*` methods: `MapControllers()` for [[Attribute Routing|attribute-routed]] controllers, `MapDefaultControllerRoute()` and `MapControllerRoute()` for conventional routes, `MapRazorPages()` for Razor Pages, and `MapGet()`/`MapPost()`/etc. for minimal APIs. `MapGroup()` (.NET 7+) organizes endpoints under shared prefixes with shared metadata. Every endpoint carries a metadata collection that middleware can query via `context.GetEndpoint().Metadata`.
>
> **Endpoint metadata** -- applied via fluent methods (`RequireAuthorization()`, `WithName()`, `WithTags()`) on minimal APIs or via attributes (`[Authorize]`, `[EnableCors]`) on controllers -- drives cross-cutting behavior without coupling it to business logic. This pattern (attribute metadata + middleware inspection) is the extensibility model for all cross-cutting concerns.
>
> **Fallback endpoints** (`MapFallbackToFile("index.html")`) catch unmatched requests, essential for SPAs. **Health check endpoints** (`MapHealthChecks()`) integrate with container orchestrators. Both leverage the same endpoint routing system with the same metadata capabilities. See [[Routing Overview]] for foundational concepts, [[Attribute Routing]] for controller-level routing, and [[Route Constraints and Tokens]] for parameter filtering.

---

## Related Topics

- [[Routing Overview]]
- [[Attribute Routing]]
- [[Route Constraints and Tokens]]
- [[Minimal APIs]]
- [[Middleware Pipeline]]
- [[Authentication and Authorization]]
- [[CORS in ASP.NET Core]]

---

## Further Reading

- [[Rate Limiting in ASP.NET Core]] -- `RequireRateLimiting` and rate limiter middleware
- [[Output Caching]] -- `CacheOutput` on endpoints
- [[OpenAPI and Swagger]] -- how endpoint metadata drives API documentation
- [[gRPC in ASP.NET Core]] -- `MapGrpcService<T>()` integration
- [[SignalR]] -- `MapHub<T>()` integration
