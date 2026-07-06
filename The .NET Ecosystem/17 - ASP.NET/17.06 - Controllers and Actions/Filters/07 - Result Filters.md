---
tags:
  - csharp
  - asp-net-core
  - filters
  - controllers
  - cross-cutting
---


Result filters run before and after the **action result** (the `IActionResult`) executes. The result is the thing that actually writes the HTTP response (e.g., `JsonResult`, `ViewResult`, `StatusCodeResult`).

- **Interface**: `IResultFilter` / `IAsyncResultFilter`
- `OnResultExecuting` -- can modify or replace the result before it writes to the response
- `OnResultExecuted` -- runs after the response body has been written (limited ability to change anything)
- Use cases: adding response headers, wrapping response envelopes, post-response logging

### Adding Custom Response Headers

```csharp
public class ResponseHeaderFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        context.HttpContext.Response.Headers["X-Request-Id"] =
            context.HttpContext.TraceIdentifier;

        context.HttpContext.Response.Headers["X-Api-Version"] = "2.0";
    }

    public void OnResultExecuted(ResultExecutedContext context)
    {
        // Response has already been sent; limited modification possible.
        // Useful for logging or cleanup.
    }
}
```

```ad-attention
You must add response headers in `OnResultExecuting`, **before** the result writes to the response body. Once the response starts streaming, headers cannot be modified. Attempting to set headers in `OnResultExecuted` will throw if the response has already started.
```
