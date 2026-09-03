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

It represents a function that can process an HTTP request — it takes only **one argument** (`HttpContext`) and returns a `Task`. It is essentially a `Func<HttpContext, Task>`. Every middleware component in the pipeline is ultimately represented as a `RequestDelegate`.

#### Why It Exists

Your handler code (a lambda, a method) knows nothing about HTTP. It takes parameters like `int id` and returns objects like `Fruit`. It cannot read from `HttpContext.Request` or write to `HttpContext.Response` on its own.

The `RequestDelegate` is the bridge between the **HTTP world** (`HttpContext`) and **your code** (lambdas, objects):

```
HTTP world (HttpContext)  ←→  RequestDelegate  ←→  Your code (lambdas, return values)
```

It pulls data **out** of `HttpContext` into your handler's parameters, calls your function, then pushes your return value **back into** `HttpContext.Response`. Without it, you would have to manually parse route values, query strings, and headers, then manually serialize and write the response for every single endpoint.

#### Two Contexts Where RequestDelegate Appears

**1. Middleware** — each middleware in the pipeline is a `RequestDelegate`. It receives `HttpContext`, does work, optionally calls the next middleware, and returns.

**2. Endpoint handlers** — when you call `MapGet`/`MapPost`/etc., the framework uses `RequestDelegateFactory` to wrap your handler into a `RequestDelegate` at startup. This generated delegate is stored in the routing table and invoked by the endpoint middleware at request time.

In both cases the signature is the same — `Task(HttpContext)` — but the way they are created differs. Middleware delegates are built from `app.Use`/`app.Run`. Endpoint delegates are built by `RequestDelegateFactory` from your handler lambda.

#### Not the Same as Your app.Use Lambda

This is **not** the same as the lambda you pass to `app.Use`:

```csharp
// This lambda is Func<HttpContext, Func<Task>, Task> — NOT a RequestDelegate.
// It has two parameters: context and next.
app.Use(async (context, next) =>
{
    // do work before
    await next();    // next is Func<Task>, not RequestDelegate
    // do work after
});
```

`app.Use` takes your two-parameter lambda and wraps it into a single `RequestDelegate` for the pipeline. The `next` parameter is a `Func<Task>` that internally invokes the next middleware's `RequestDelegate` with the current `HttpContext`.

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

    await next();  // next is Func<Task>, invokes the rest of the pipeline

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

### RequestDelegateFactory — The Bridge Between Your Handler and the Pipeline

When you register an endpoint with `MapGet`, `MapPost`, etc., the framework needs to turn your handler (a simple lambda or method) into a `RequestDelegate` that the pipeline can invoke. **`RequestDelegateFactory`** does this at startup.

Your handler knows nothing about HTTP — it just takes parameters and returns an object:

```csharp
app.MapGet("/fruit/{id}", (int id) => new Fruit("Apple", 10));
```

`RequestDelegateFactory.Create()` wraps it into a `RequestDelegate` that handles all the plumbing:

```csharp
// What the framework generates (simplified):
RequestDelegate generated = async (HttpContext context) =>
{
    // 1. Bind parameters from the request
    var id = int.Parse(context.Request.RouteValues["id"]!.ToString()!);

    // 2. Call your handler
    var result = yourHandler(id);

    // 3. Write the return value to the response
    await context.Response.WriteAsJsonAsync(result);
};
```

Without `RequestDelegateFactory`, you would have to write this plumbing yourself for every endpoint.

### How the Return Value Is Handled

The generated `RequestDelegate` handles your return value differently depending on its type:

| Handler returns    | What the generated delegate does             |
| ------------------ | -------------------------------------------- |
| Plain object       | Serializes to JSON via `System.Text.Json`    |
| `string`           | Writes as `text/plain`                       |
| `IResult`          | Calls `result.ExecuteAsync(context)`         |
| `void` / `Task`    | Returns 200 with empty body                  |

When your handler returns an `IResult`, the generated delegate does **not** serialize it — it delegates to the `IResult` object's own `ExecuteAsync` method, which writes the response itself:

```csharp
// If handler returns IResult:
RequestDelegate generated = async (HttpContext context) =>
{
    var id = int.Parse(context.Request.RouteValues["id"]!.ToString()!);
    IResult result = yourHandler(id);  // e.g. Results.Ok(fruit) or Results.NotFound()
    await result.ExecuteAsync(context); // IResult writes its own response
};
```

Each `IResult` implementation knows how to write itself:

```csharp
// Ok<T> sets status 200 and serializes the value
// NotFound sets status 404 with no body
// BadRequest<T> sets status 400 and serializes the error
```

### Connecting the Dots: RequestDelegateFactory → RequestDelegate → ExecuteAsync

```
STARTUP
───────
MapGet("/fruit/{id}", handler)
    │
    ▼
RequestDelegateFactory.Create()
    "Wrap this handler into a RequestDelegate"
    │
    ▼
RequestDelegate is stored in the routing table

REQUEST TIME
────────────
HTTP Request arrives
    → Kestrel creates HttpContext
    → Routing middleware matches URL to endpoint
    → Endpoint middleware calls the stored RequestDelegate
        → RequestDelegate binds parameters from HttpContext
        → RequestDelegate calls your handler
        → If plain object → serializes to JSON → writes to HttpContext.Response
        → If IResult → calls result.ExecuteAsync(context)
    → Kestrel sends response bytes over HTTP
```

- **`RequestDelegateFactory`** runs once at **startup** — it builds the `RequestDelegate`
- **`RequestDelegate`** runs at **request time** — it is the wrapper that binds parameters, calls your handler, and writes the response
- **`ExecuteAsync`** runs at **request time** — but only when your handler returns an `IResult`, allowing the result object to write its own response

> [!summary] Section Summary
> `RequestDelegate` is a delegate that takes a single `HttpContext` and returns `Task` — it represents one step in the pipeline. The two-parameter lambda in `app.Use(async (context, next) => ...)` is **not** a `RequestDelegate`; it's a `Func<HttpContext, Func<Task>, Task>` that `app.Use` wraps into one. `HttpContext` is the per-request object carrying all request and response data. `RequestDelegateFactory` bridges the gap between your simple handler and the pipeline by generating a `RequestDelegate` that handles parameter binding, handler invocation, and response writing — including calling `ExecuteAsync` on `IResult` return values.
