---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## Comprehensive Summary

> [!tip] Complete Summary
> ASP.NET Core's built-in middleware components form a layered pipeline that handles cross-cutting concerns before requests reach your application logic.
>
> **Error handling** -- `UseExceptionHandler` (production) and `UseDeveloperExceptionPage` (development) ensure exceptions are caught and handled appropriately. Never expose developer error pages in production.
>
> **Security** -- `UseHsts` and `UseHttpsRedirection` enforce HTTPS. `UseAuthentication` identifies users, `UseAuthorization` enforces access control, and `UseCors` controls cross-origin access. The order between these is critical.
>
> **Performance** -- `UseStaticFiles` short-circuits for static content, `UseResponseCaching` caches responses based on HTTP headers, `UseResponseCompression` reduces bandwidth with gzip/Brotli, and `UseRateLimiter` protects against excessive traffic.
>
> **Routing** -- `UseRouting` is the central hub that matches URLs to endpoints, enabling all metadata-aware middleware (auth, CORS, rate limiting) to function correctly.
>
> **Session** -- `UseSession` provides server-side state storage backed by distributed cache providers.
>
> The single most important principle across all middleware is **order matters**. The recommended pipeline order (exception handling, HTTPS, static files, routing, CORS, authentication, authorization, session, caching, rate limiting, endpoint execution) ensures each component has the context it needs from earlier components. Placing middleware out of order is the most common cause of subtle bugs in ASP.NET Core applications.

## Related Topics

- [[Middleware Overview]] -- foundational concepts of the request pipeline and how middleware works
- [[Request Pipeline]] -- the full request/response flow through the middleware chain
- [[Custom Middleware]] -- writing your own middleware components
- [[Authentication and Authorization]] -- deeper dive into identity and access control
- [[Routing]] -- endpoint routing internals and route templates
- [[Error Handling and Logging]] -- structured error handling strategies
- [[Dependency Injection]] -- how middleware integrates with the DI container

## Further Reading

- [[ASP.NET Core Fundamentals]] -- overview of the framework architecture
- [[Configuration]] -- managing app settings that middleware depends on
- [[Web APIs]] -- building APIs that leverage these middleware components
- [[Controllers and Actions]] -- MVC controller patterns that interact with routing and authorization
