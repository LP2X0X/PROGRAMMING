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

### Bindable Types from HttpContext

`HttpContext` contains all the details on both the request and the response. You can access everything you need from it, but often there is an easier way -- ASP.NET Core lets you bind directly to its inner types as method parameters. These are recognized and resolved automatically:

**`HttpRequest`** -- equivalent to `HttpContext.Request`. Contains all the details about the incoming request only (URL, headers, body, query string, cookies). Use this when you only need request data and don't care about the response.

**`HttpResponse`** -- equivalent to `HttpContext.Response`. Contains all the details about the outgoing response only (status code, headers, body). Use this when you need to manipulate the response directly.

**`CancellationToken`** -- equivalent to `HttpContext.RequestAborted`. This token is canceled if the client aborts the request (e.g., closes the browser tab, times out). It is useful when you need to cancel a long-running task early rather than wasting server resources on a response nobody will receive.

**`ClaimsPrincipal`** -- equivalent to `HttpContext.User`. Contains authentication information about the user -- their identity, claims, and roles. Populated by the authentication middleware.

**`Stream`** -- equivalent to `HttpRequest.Body`. A reference to the raw request body stream. Useful for scenarios where you need to process large amounts of data efficiently without holding it all in memory at the same time (e.g., file uploads, streaming payloads).

**`PipeReader`** -- equivalent to `HttpRequest.BodyReader`. Provides a higher-level API compared to `Stream` for reading the request body, built on `System.IO.Pipelines`. Useful in similar large-payload scenarios but with better buffer management and less copying.

```csharp
// CancellationToken -- cancel if client disconnects
app.MapGet("/reports/{id}", async (int id, CancellationToken ct) =>
{
    var report = await _service.GenerateAsync(id, ct);
    return Results.Ok(report);
});

// Stream -- process large payload without buffering
app.MapPost("/upload", async (Stream body) =>
{
    await ProcessLargePayloadAsync(body);
    return Results.Ok();
});
```

> [!ad-note]
> Prefer binding the specific type you need rather than the full `HttpContext`. It makes dependencies explicit, simplifies testing, and signals exactly what data the handler uses.

> [!warning] Common Misconception
> `HttpContext` is **not** thread-safe. Do not store a reference to it and access it from a background thread. If you need to use request data in a background task, copy the values you need into local variables first.

> [!summary] Section Summary
> `RequestDelegate` is the delegate type (`Task RequestDelegate(HttpContext)`) that represents each step in the pipeline. `HttpContext` is the per-request object carrying all request and response data. Together, they form the core abstraction: each middleware receives an `HttpContext` and a `RequestDelegate` pointing to the next middleware.
