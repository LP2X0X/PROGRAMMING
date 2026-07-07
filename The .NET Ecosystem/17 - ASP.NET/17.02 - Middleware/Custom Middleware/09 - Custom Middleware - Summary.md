---
tags:
  - csharp
  - asp-net-core
  - middleware
  - custom
---

## Comprehensive Summary

> [!tip] Complete Summary
> **Custom middleware** extends the ASP.NET Core [[Request Pipeline]] with user-defined logic that runs for every HTTP request.
>
> **Three approaches exist:**
> 1. **Inline** (`app.Use()`) -- quick lambdas for prototyping
> 2. **Convention-based** -- a class with `RequestDelegate` constructor and `InvokeAsync` method; singleton lifetime; the standard production approach
> 3. **Factory-based** (`IMiddleware`) -- resolved from DI per request; supports scoped constructor injection; requires explicit DI registration
>
> **Dependency injection** in convention-based middleware splits into constructor injection (singleton services only) and method injection via `InvokeAsync` parameters (any lifetime). Injecting scoped services like `DbContext` into the constructor is a common bug -- the singleton middleware captures a single instance that outlives its scope.
>
> **The extension method pattern** (`app.UseRequestTiming()`) wraps `UseMiddleware<T>()` for clean, discoverable registration following ASP.NET Core conventions.
>
> **Real-world examples** include request timing (Stopwatch + `OnStarting` callback), correlation ID propagation (check/generate/echo header), API key validation (constant-time comparison, short-circuit on failure), and global exception handling (try-catch + ProblemDetails JSON response).
>
> **Testing** uses `DefaultHttpContext` and a mock `RequestDelegate` lambda. Call `Response.StartAsync()` to trigger `OnStarting` callbacks in unit tests. Use `WebApplicationFactory<T>` for integration tests.
>
> **Middleware vs action filters**: middleware runs at the HTTP pipeline level for all requests; action filters run inside the MVC pipeline for controller actions only. Use middleware for cross-cutting infrastructure concerns, action filters for MVC-specific logic.

## Related Topics

- [[Middleware Overview]] -- how the ASP.NET Core request pipeline works
- [[Request Pipeline]] -- middleware ordering, branching with `Map`, and terminal middleware
- [[Built-in Middleware]] -- authentication, CORS, static files, response compression
- [[Service Lifetimes]] -- singleton, scoped, transient and how they interact with middleware
- [[The .NET Ecosystem/17 - ASP.NET/17.03 - Dependency Injection/DI Overview/DI Overview]] -- the ASP.NET Core dependency injection container
- [[Error Handling and Logging]] -- structured logging, exception pages, ProblemDetails

## Further Reading

- [[Action Filters]] -- MVC-specific request/response interception
- [[Minimal APIs]] -- middleware works with minimal API endpoints too
- [[Options Pattern]] -- configuring middleware with `IOptions<T>`
- [[Integration Testing]] -- using `WebApplicationFactory<T>` to test the full pipeline
- [[Response Caching]] -- built-in middleware for HTTP caching
- [[Rate Limiting]] -- built-in rate limiting middleware in .NET 7+
