---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
---

## Comparison to ASP.NET HTTP Modules and Handlers

If you have experience with classic ASP.NET (System.Web), it helps to understand how the old model maps to the new middleware model.

| Aspect | ASP.NET (Classic) HTTP Modules/Handlers | ASP.NET Core Middleware |
|---|---|---|
| **Concept** | Separate Modules (cross-cutting) and Handlers (endpoint) | Unified middleware components for both concerns |
| **Configuration** | XML in `web.config` | Code in `Program.cs` / `Startup.cs` |
| **Lifecycle** | Fixed event-based lifecycle (BeginRequest, AuthenticateRequest, etc.) | Flexible delegate chain -- you control the order |
| **Execution model** | Event-driven: modules subscribe to specific pipeline events | Sequential delegate chain: each calls `next()` |
| **Dependency injection** | Not built-in; requires workarounds | First-class DI support via constructor injection |
| **Testability** | Difficult to unit test (tightly coupled to HttpApplication) | Easy to test (pass in mock HttpContext) |
| **Granularity** | Modules are global; Handlers are per-extension/route | Middleware is per-request with full control over branching |
| **Async support** | Limited; async modules added later with complexity | Fully async from the ground up (`Task`-based) |
| **Hosting** | IIS-only | Cross-platform (Kestrel, IIS, Nginx, Docker) |
| **Short-circuiting** | Requires `CompleteRequest()` -- stops events but still runs EndRequest | Simply do not call `next()` -- clean and predictable |

> [!ad-note]
> The classic ASP.NET pipeline had a fixed set of events (BeginRequest, AuthenticateRequest, AuthorizeRequest, etc.) that modules could hook into. ASP.NET Core replaced this rigid system with a simple, composable delegate chain that gives you complete control over the order and behavior of pipeline components.

> [!summary] Section Summary
> The classic ASP.NET model used HTTP Modules (for cross-cutting concerns) and HTTP Handlers (for endpoints) configured via XML and tied to a fixed event lifecycle. ASP.NET Core replaced this with a unified, flexible middleware pipeline that is code-configured, fully async, supports DI natively, and runs cross-platform.
