---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
  - request
---

## Common Misconceptions

> [!warning] Common Misconception
> **"Middleware order does not matter because ASP.NET Core figures it out."** This is false. ASP.NET Core executes middleware in exactly the order you register it. There is no automatic reordering. If you put authorization before authentication, authorization will always see an unauthenticated user.

> [!warning] Common Misconception
> **"UseStaticFiles after UseAuthorization protects my static files."** Static file middleware does not participate in the authorization system. It checks for file existence on disk and serves the file regardless of authorization status. To protect static files, serve them through a controller action or use `MapStaticAssets()` with authorization.

> [!warning] Common Misconception
> **"I can call UseRouting() and UseEndpoints() multiple times to create multiple pipelines."** While technically possible, each call to `UseRouting()` starts a new routing scope. This is rarely needed and usually indicates a misunderstanding. Use `Map()` branching instead for separate pipelines.

> [!warning] Common Misconception
> **"UseRouting() is not needed because MapControllers() handles routing."** In modern ASP.NET Core (6+), `UseRouting()` is called implicitly. However, if you need middleware to access endpoint metadata BEFORE authentication runs (such as a custom CORS handler), you must call `UseRouting()` explicitly to control where the routing phase occurs in the pipeline.

> [!summary] Section Summary
> The most dangerous misconceptions involve assuming middleware ordering is automatic, believing static files are protected by authorization middleware, and not understanding that `UseRouting()` is implicitly called but can be made explicit when precise ordering control is needed.
