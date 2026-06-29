---
tags:
  - csharp
  - asp-net-core
  - filters
  - controllers
  - cross-cutting
---


Understanding when to use filters versus [[17.02 - Middleware]] is important. They serve overlapping but distinct roles.

| Aspect | Middleware | Filters |
|---|---|---|
| Scope | Every HTTP request | Only MVC/Razor Pages requests |
| Context | `HttpContext` only | `ActionContext`, `ModelState`, controller, arguments |
| Ordering | Pipeline order in `Program.cs` | Type-based + scope-based + `Order` |
| Short-circuit | Return without calling `next()` | Set `context.Result` |
| DI | Constructor injection | Constructor injection (via TypeFilter/ServiceFilter) |

### When to Use Middleware

- Authentication / authorization schemes
- CORS
- Static file serving
- Request logging (all requests, including non-MVC)
- Global exception handling (`UseExceptionHandler`)
- Request/response compression
- Rate limiting

### When to Use Filters

- Action-specific authorization policies
- Model validation logic
- Action-specific logging (with access to action arguments)
- Response modification (headers, wrapping)
- MVC-specific exception handling
- Caching for specific actions

```ad-tip
A good rule of thumb: if you need access to MVC-specific information (action name, model state, action arguments, controller instance), use a filter. If the concern applies to all HTTP traffic regardless of whether MVC handles it, use middleware.
```
