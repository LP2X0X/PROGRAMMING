---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
---

## Practical Tips

> [!tip] Tip: Keep Middleware Focused
> Each middleware should do **one thing**. If your middleware is handling logging, authentication, AND response transformation, split it into three separate middleware components. This makes each piece testable, reorderable, and replaceable.

> [!tip] Tip: Use Short-Circuiting Wisely
> Short-circuiting (not calling `next()`) is powerful but must be used carefully. Always set the response status code and write a response body before short-circuiting, otherwise the client receives an empty response with a 200 status.

> [!tip] Tip: Do Not Modify Response Headers After WriteAsync
> Once you call `context.Response.WriteAsync()` or start writing to the response body, the headers are sent to the client. Any attempt to modify headers after that point throws an `InvalidOperationException`. Use `context.Response.OnStarting()` to set headers just before they are sent.

```csharp
// WRONG: Headers modified after response body is started
app.Use(async (context, next) =>
{
    await next();
    // If the endpoint already wrote to the response, this THROWS:
    context.Response.Headers["X-Custom"] = "value";  // InvalidOperationException!
});

// CORRECT: Use OnStarting to guarantee header modification before send
app.Use(async (context, next) =>
{
    context.Response.OnStarting(() =>
    {
        context.Response.Headers["X-Custom"] = "value";
        return Task.CompletedTask;
    });

    await next();
});
```

> [!tip] Tip: Use Items for Per-Request Data Sharing
> Use `context.Items` to pass data between middleware components within a single request. Do not use static fields or singletons for per-request state -- those are shared across all requests and cause race conditions.

> [!summary] Section Summary
> Keep middleware focused on a single concern. Always set status codes when short-circuiting. Use `OnStarting` to safely modify response headers. Share per-request data through `context.Items`, not static state.
