---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
---

## Comprehensive Summary

> [!tip] Complete Summary
> **Middleware** is the core mechanism for processing HTTP requests in ASP.NET Core. Each middleware component sits in a pipeline, receives an `HttpContext`, and decides whether to pass the request to the next component via `next()` or short-circuit the pipeline.
>
> **Three fundamental methods** build the pipeline: `app.Use()` for chainable middleware that calls `next()`, `app.Run()` for terminal middleware that ends the pipeline, and `app.Map()` for branching the pipeline by path.
>
> The pipeline follows the **onion model**: code before `await next()` runs on the way in (request phase), and code after runs on the way out (response phase, in reverse order). This means the first registered middleware is the first to see the request and the last to see the response.
>
> **Order is critical** and causes functional bugs when wrong. The standard order is: Exception Handling, HTTPS Redirection, Static Files, Routing, CORS, Authentication, Authorization, Custom Middleware, Endpoints. Authentication must come before Authorization. CORS must handle preflight before Authorization rejects it. Exception handling must be outermost to catch all errors.
>
> The two key types are `RequestDelegate` (the delegate representing each pipeline step) and `HttpContext` (the per-request object carrying all request/response data).
>
> Compared to classic ASP.NET's rigid HTTP Modules/Handlers with XML configuration and fixed event lifecycles, ASP.NET Core middleware is a flexible, code-configured, fully async delegate chain with first-class DI support and cross-platform hosting.

## Related Topics

- [[Request Pipeline]] -- detailed breakdown of the full ASP.NET Core request pipeline
- [[Custom Middleware]] -- writing middleware as a class with constructor injection
- [[Built-in Middleware]] -- reference for all built-in middleware components (CORS, Auth, Static Files, etc.)
- [[Dependency Injection in ASP.NET Core]] -- how services are resolved in middleware
- [[Exception Handling Middleware]] -- global error handling patterns
- [[Authentication and Authorization]] -- the auth middleware duo in depth
- [[Minimal APIs]] -- how middleware works with the minimal API model

## Further Reading

- [[Filters vs Middleware]] -- when to use MVC filters instead of middleware
- [[Endpoint Routing]] -- how routing middleware resolves endpoints
- [[IApplicationBuilder]] -- the interface behind `app.Use()`, `app.Run()`, and `app.Map()`
- [[Middleware Testing]] -- unit testing middleware with `DefaultHttpContext`
- [[Response Caching Middleware]] -- caching responses at the middleware level
- [[Custom Middleware]] -- convention-based and factory-based approaches
