---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---


## Related Topics

- [[Middleware Overview]] -- how middleware works and why exception handling must be the outermost layer
- [[ILogger and Logging]] -- structured logging for exception details and diagnostics
- [[Problem Details]] -- the RFC 9457 standard format for API error responses
- [[Filters vs Middleware]] -- deeper comparison of MVC filters and middleware
- [[Authentication and Authorization]] -- exception handling for auth-related errors

---

## Further Reading

- [[Custom Middleware]] -- writing class-based middleware with constructor injection
- [[Health Checks]] -- monitoring application health alongside error handling
- [[API Versioning]] -- consistent error responses across API versions

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Exception handling in ASP.NET Core** operates at the middleware pipeline level. The framework provides built-in middleware for two primary scenarios: the **Developer Exception Page** for development (rich stack traces, source code, request details) and **`UseExceptionHandler`** for production (catches exceptions, re-executes the pipeline to an error path without browser redirects).
>
> For non-exception error status codes (404, 403), **Status Code Pages** middleware fills the gap -- `UseStatusCodePagesWithReExecute` is preferred over redirects because it preserves the original URL and returns the correct status code.
>
> **Custom exception handling middleware** provides full control when applications serve both MVC views and API endpoints, using content negotiation to return HTML error pages or JSON [[Problem Details]] as appropriate.
>
> **.NET 8 introduced `IExceptionHandler`**, the modern approach that supports multiple handlers in priority order with full DI support. Handlers are tried sequentially -- the first to return `true` stops the chain. Register specific handlers (validation, domain exceptions) before the catch-all handler.
>
> **Exception filters** (`IExceptionFilter`) are MVC-specific and only catch exceptions from controller actions. They complement but do not replace exception middleware, which catches everything including exceptions from other middleware and minimal APIs.
>
> **Domain exception types** (NotFoundException, ValidationException, etc.) should carry their own HTTP status code and error code, enabling clean pattern matching in the exception handler. Log 4xx as Warning and 5xx as Error to avoid alert fatigue.
>
> The cardinal rule: **never expose internal details in production**. Return generic messages with a trace ID that maps to the full exception in your server logs. The Developer Exception Page must only run in the Development environment.
>
> **Controller-level try-catch** should be reserved for graceful degradation and recovery scenarios -- not for translating exceptions into error responses, which is the global handler's responsibility.
