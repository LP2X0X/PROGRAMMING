---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
---

## RequestDelegate and HttpContext

The two most important types in the middleware system are **`RequestDelegate`** and **`HttpContext`**.

### RequestDelegate

**`RequestDelegate`** is a delegate type defined as:

```csharp
public delegate Task RequestDelegate(HttpContext context);
```

It represents a function that can process an HTTP request. Every middleware component ultimately works with `RequestDelegate` -- it receives one (representing the next middleware) and produces one (representing itself plus the rest of the pipeline).

When you call `await next()` inside middleware, you are invoking the `RequestDelegate` that represents the remainder of the pipeline.

### HttpContext

**`HttpContext`** is the object that encapsulates all information about the current HTTP request and response. It is created per-request and flows through the entire pipeline.

Key properties of `HttpContext`:

| Property | Type | Description |
|---|---|---|
| `Request` | `HttpRequest` | The incoming HTTP request (URL, headers, body, query string) |
| `Response` | `HttpResponse` | The outgoing HTTP response (status code, headers, body) |
| `User` | `ClaimsPrincipal` | The authenticated user (populated by authentication middleware) |
| `Items` | `IDictionary<object, object>` | Per-request storage for sharing data between middleware |
| `RequestServices` | `IServiceProvider` | The DI container scoped to this request |
| `Connection` | `ConnectionInfo` | Client connection details (IP address, port, SSL) |
| `Features` | `IFeatureCollection` | Low-level server features |

```csharp
app.Use(async (context, next) =>
{
    // Reading from HttpContext.Request
    string method = context.Request.Method;           // "GET", "POST", etc.
    string path = context.Request.Path;               // "/api/orders"
    string? authHeader = context.Request.Headers["Authorization"];
    string? orderId = context.Request.Query["orderId"];

    // Sharing data between middleware via Items
    context.Items["RequestStartTime"] = DateTime.UtcNow;

    await next();

    // Reading from HttpContext.Response (after endpoint has set it)
    int statusCode = context.Response.StatusCode;     // 200, 404, 500, etc.

    // Retrieving shared data
    var startTime = (DateTime)context.Items["RequestStartTime"]!;
    var elapsed = DateTime.UtcNow - startTime;
    Console.WriteLine($"{method} {path} -> {statusCode} ({elapsed.TotalMilliseconds}ms)");
});
```

> [!warning] Common Misconception
> `HttpContext` is **not** thread-safe. Do not store a reference to it and access it from a background thread. If you need to use request data in a background task, copy the values you need into local variables first.

> [!summary] Section Summary
> `RequestDelegate` is the delegate type (`Task RequestDelegate(HttpContext)`) that represents each step in the pipeline. `HttpContext` is the per-request object carrying all request and response data. Together, they form the core abstraction: each middleware receives an `HttpContext` and a `RequestDelegate` pointing to the next middleware.
