---
tags:
  - csharp
  - asp-net-core
  - http
  - status-codes
---

## Quick Reference Table

A consolidated reference of every common HTTP status code, its meaning, and the ASP.NET Core helper method (if applicable).

| Code | Name | Category | Meaning | ASP.NET Core Helper |
|---|---|---|---|---|
| 100 | Continue | Informational | Send the body | *(automatic)* |
| 101 | Switching Protocols | Informational | Upgrading to WebSocket | *(automatic)* |
| **200** | OK | Success | Standard success | `Ok()` / `Ok(value)` |
| **201** | Created | Success | Resource created | `CreatedAtAction()` / `CreatedAtRoute()` |
| **202** | Accepted | Success | Queued for processing | `Accepted()` |
| **204** | No Content | Success | Success, no body | `NoContent()` |
| **301** | Moved Permanently | Redirect | URL permanently changed | `RedirectPermanent()` |
| **302** | Found | Redirect | Temporary redirect | `Redirect()` / `RedirectToAction()` |
| **304** | Not Modified | Redirect | Use cached version | *(response caching middleware)* |
| **307** | Temporary Redirect | Redirect | Temp redirect, keep method | `RedirectPreserveMethod()` |
| **400** | Bad Request | Client Error | Malformed / invalid data | `BadRequest()` |
| **401** | Unauthorized | Client Error | Not authenticated | `Unauthorized()` / `Challenge()` |
| **403** | Forbidden | Client Error | Authenticated, no permission | `Forbid()` |
| **404** | Not Found | Client Error | Resource does not exist | `NotFound()` |
| **405** | Method Not Allowed | Client Error | Wrong HTTP method | *(automatic by routing)* |
| **409** | Conflict | Client Error | State conflict | `Conflict()` |
| **415** | Unsupported Media Type | Client Error | Wrong Content-Type | *(automatic by `[ApiController]`)* |
| **422** | Unprocessable Entity | Client Error | Semantically invalid | `UnprocessableEntity()` |
| **429** | Too Many Requests | Client Error | Rate limited | *(rate limiting middleware)* |
| **500** | Internal Server Error | Server Error | Unhandled exception | `StatusCode(500)` *(via middleware)* |
| **502** | Bad Gateway | Server Error | Upstream error | `StatusCode(502)` |
| **503** | Service Unavailable | Server Error | Server overloaded/down | `StatusCode(503)` |
| **504** | Gateway Timeout | Server Error | Upstream timeout | `StatusCode(504)` |

> [!ad-tip] Using `StatusCode()` for Any Code
> For status codes that do not have a dedicated helper method, use the generic `StatusCode()` method:
> ```csharp
> return StatusCode(418); // I'm a teapot
> return StatusCode(429, new { error = "Rate limit exceeded", retryAfter = 60 });
> return StatusCode(503, new { error = "Service temporarily unavailable" });
> ```

> [!summary] Section Summary
> - Most common codes have dedicated ASP.NET Core helper methods (`Ok()`, `NotFound()`, `BadRequest()`, etc.)
> - Some codes are returned automatically by the framework (`405`, `415`, `304`)
> - Use `StatusCode(int)` for any code that lacks a dedicated helper
> - The table above covers the codes you will encounter in 99% of web development
