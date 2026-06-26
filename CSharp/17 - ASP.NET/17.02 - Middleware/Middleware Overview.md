---
tags: [csharp, asp-net-core, middleware, pipeline]
aliases: [ASP.NET Core Middleware, Middleware Pipeline, Request Pipeline Middleware]
status: complete
date: 2026-06-18
---

# Middleware Overview

**Middleware** is a fundamental building block in ASP.NET Core that forms the **request processing pipeline**. Every HTTP request that enters your application flows through a series of middleware components, each of which can inspect, modify, or short-circuit the request before it reaches your application logic -- and do the same on the way back out with the response.

Understanding middleware is essential because it controls everything that happens to a request: logging, authentication, authorization, error handling, routing, CORS, compression, and more. If something goes wrong in your pipeline, it is almost always a middleware ordering issue.

---

## Table of Contents

- [[#What Is Middleware]]
- [[#The Request Pipeline -- Delegate Chain]]
- [[#The Onion Model -- Execution Order]]
- [[#RequestDelegate and HttpContext]]
- [[#The Three Fundamental Methods]]
- [[#Inline Middleware with app.Use]]
- [[#Why Order Matters]]
- [[#Comparison to ASP.NET HTTP Modules and Handlers]]
- [[#Practical Tips]]
- [[#Related Topics]]
- [[#Further Reading]]
- [[#Comprehensive Summary]]

---

## What Is Middleware

A **middleware component** is a piece of code that sits in the HTTP request pipeline and has access to both the incoming `HttpRequest` and the outgoing `HttpResponse`. Each middleware component can:

1. **Inspect** the request (read headers, check authentication tokens, log information)
2. **Modify** the request or response (add headers, transform the body, set status codes)
3. **Pass the request** to the next middleware in the pipeline by calling `next()`
4. **Short-circuit** the pipeline by NOT calling `next()`, which means no further middleware runs

Think of middleware as a series of checkpoints at an airport. Each checkpoint examines you, possibly stamps your passport, and either lets you proceed to the next checkpoint or turns you away. On your way back out, you pass through the same checkpoints in reverse.

```csharp
// A simple middleware that logs request timing
app.Use(async (context, next) =>
{
    var stopwatch = System.Diagnostics.Stopwatch.StartNew();
    
    // Code here runs BEFORE the next middleware (on the way IN)
    Console.WriteLine($"[Request] {context.Request.Method} {context.Request.Path}");
    
    await next();  // Pass control to the next middleware
    
    // Code here runs AFTER the next middleware (on the way OUT)
    stopwatch.Stop();
    Console.WriteLine($"[Response] {context.Response.StatusCode} in {stopwatch.ElapsedMilliseconds}ms");
});
```

> [!info] Definition
> **Middleware** = a component assembled into an application pipeline to handle requests and responses. Each component chooses whether to pass the request to the next component and can perform work before and after the next component in the pipeline.

> [!summary] Section Summary
> Middleware components are the building blocks of the ASP.NET Core request pipeline. Each component can inspect, modify, pass along, or short-circuit HTTP requests and responses. They execute in a defined order, giving you full control over how requests are processed.

---

## The Request Pipeline -- Delegate Chain

The ASP.NET Core request pipeline is built as a **chain of request delegates**. Each delegate represents one middleware component. When a request arrives, the framework invokes the first delegate, which optionally invokes the next, and so on -- forming a chain.

Here is an ASCII diagram showing how a request flows through three middleware components and back:

```
                        ASP.NET Core Request Pipeline

  HTTP Request
       |
       v
  +--------------------+
  | Middleware 1        |
  |  (Logging)          |
  |                     |
  |  Before next() -----+--->  +--------------------+
  |                     |      | Middleware 2        |
  |                     |      |  (Authentication)   |
  |                     |      |                     |
  |                     |      |  Before next() -----+--->  +--------------------+
  |                     |      |                     |      | Middleware 3        |
  |                     |      |                     |      |  (Routing/Endpoint) |
  |                     |      |                     |      |                     |
  |                     |      |                     |      |  Generates Response |
  |                     |      |                     |      |                     |
  |                     |      |  After next()  <----+------|  (terminal)         |
  |                     |      |                     |      +--------------------+
  |  After next()  <----+------+                     |
  |                     |      +--------------------+
  +--------------------+
       |
       v
  HTTP Response
```

And here is a simplified linear view:

```
  Request  --->  [Logging] ---> [Auth] ---> [CORS] ---> [Routing] ---> [Endpoint]
  Response <---  [Logging] <--- [Auth] <--- [CORS] <--- [Routing] <--- [Endpoint]
```

Each arrow marked `next()` is a call to the **next delegate** in the chain. The last middleware (often the endpoint/routing middleware) generates the actual response, and then control flows back through each middleware in reverse order.

> [!ad-note]
> The pipeline is constructed at application startup in `Program.cs` (or `Startup.cs` in older projects). Once built, the delegate chain is immutable -- it does not change per request.

> [!summary] Section Summary
> The request pipeline is a chain of delegates where each middleware calls `next()` to invoke the next one. The request flows forward through the chain, and the response flows backward. The chain is built at startup and remains fixed for the lifetime of the application.

---

## The Onion Model -- Execution Order

The way middleware executes is often described as the **"onion model"** or **"Russian doll" model**. Each middleware wraps around the next one, like layers of an onion. Code before `await next()` runs on the way in (request phase), and code after `await next()` runs on the way out (response phase).

```
                         The Onion Model
  
             +---------------------------------------+
             |          Middleware 1 (outer)          |
             |   +-------------------------------+   |
             |   |      Middleware 2 (middle)     |   |
             |   |   +-----------------------+   |   |
             |   |   |   Middleware 3 (inner) |   |   |
             |   |   |                       |   |   |
  Request ===+===+===+===>  Endpoint    ====>+===+===+===> Response
             |   |   |                       |   |   |
             |   |   +-----------------------+   |   |
             |   +-------------------------------+   |
             +---------------------------------------+

  Execution Order:
    IN:   Middleware 1 -> Middleware 2 -> Middleware 3 -> Endpoint
    OUT:  Middleware 3 -> Middleware 2 -> Middleware 1 -> Client
```

Here is a concrete example demonstrating the execution order:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.Use(async (context, next) =>
{
    Console.WriteLine("1. Middleware A - BEFORE next()");
    await next();
    Console.WriteLine("6. Middleware A - AFTER next()");
});

app.Use(async (context, next) =>
{
    Console.WriteLine("2. Middleware B - BEFORE next()");
    await next();
    Console.WriteLine("5. Middleware B - AFTER next()");
});

app.Use(async (context, next) =>
{
    Console.WriteLine("3. Middleware C - BEFORE next()");
    await next();
    Console.WriteLine("4. Middleware C - AFTER next()");
});

app.Run(async context =>
{
    Console.WriteLine("   >>> Endpoint reached <<<");
    await context.Response.WriteAsync("Hello from the endpoint");
});

app.Run();
```

**Console output for a single request:**

```
1. Middleware A - BEFORE next()
2. Middleware B - BEFORE next()
3. Middleware C - BEFORE next()
   >>> Endpoint reached <<<
4. Middleware C - AFTER next()
5. Middleware B - AFTER next()
6. Middleware A - AFTER next()
```

Notice how the "AFTER" messages print in **reverse order**. This is the onion model in action -- the outermost layer is first to see the request and last to see the response.

> [!warning] Common Misconception
> Many beginners think middleware runs top-to-bottom and then stops. In reality, each middleware runs **twice** -- once on the way in (before `next()`) and once on the way out (after `next()`). If you place response-modifying code before `next()`, it may be overwritten by a later middleware. If you place request-reading code after `next()`, the request may have already been consumed.

> [!summary] Section Summary
> Middleware follows the onion model: each component wraps around the next. Code before `await next()` executes during the request phase (outside-in), and code after `await next()` executes during the response phase (inside-out, reverse order). This two-phase execution is critical to understand for correct middleware behavior.

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

---

## The Three Fundamental Methods

ASP.NET Core provides three fundamental methods for adding middleware to the pipeline: **`app.Use()`**, **`app.Run()`**, and **`app.Map()`**. Understanding the differences between them is essential.

### app.Use() -- Chainable Middleware

**`app.Use()`** adds a middleware that **can call `next()`** to pass the request to the next middleware. It is the most common method and is used for middleware that needs to do work both before and after subsequent middleware.

```csharp
// app.Use() -- always call next() unless you want to short-circuit
app.Use(async (context, next) =>
{
    // Check for an API key on all requests
    if (!context.Request.Headers.ContainsKey("X-Api-Key"))
    {
        context.Response.StatusCode = 401;
        await context.Response.WriteAsync("API key is required");
        return;  // Short-circuit: do NOT call next()
    }

    // API key present -- continue to next middleware
    await next();
});
```

### app.Run() -- Terminal Middleware

**`app.Run()`** adds a **terminal middleware** -- it does **not** receive a `next` parameter and therefore cannot pass the request further down the pipeline. It always ends the pipeline.

```csharp
// app.Run() -- terminal, no next() available
app.Run(async context =>
{
    // This is the end of the pipeline
    await context.Response.WriteAsync("Request handled by terminal middleware");
});

// WARNING: Any middleware registered after app.Run() will NEVER execute
app.Use(async (context, next) =>
{
    // This code is unreachable!
    Console.WriteLine("This will never print");
    await next();
});
```

> [!danger]
> Never place `app.Run()` before other middleware unless you intentionally want to terminate the pipeline at that point. Any middleware added after `app.Run()` is dead code and will never execute.

### app.Map() -- Branch the Pipeline

**`app.Map()`** creates a **branch** in the pipeline based on the request path. When the request path matches the specified prefix, the request is routed to a separate middleware pipeline. The matched path segment is removed from `HttpContext.Request.Path` and appended to `HttpContext.Request.PathBase`.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Main pipeline middleware
app.Use(async (context, next) =>
{
    Console.WriteLine("Main pipeline: before branch");
    await next();
    Console.WriteLine("Main pipeline: after branch");
});

// Branch for /api requests
app.Map("/api", apiApp =>
{
    apiApp.Use(async (context, next) =>
    {
        Console.WriteLine($"API branch: handling {context.Request.Path}");
        await next();
    });

    apiApp.Run(async context =>
    {
        await context.Response.WriteAsync("API response");
    });
});

// Branch for /health requests
app.Map("/health", healthApp =>
{
    healthApp.Run(async context =>
    {
        await context.Response.WriteAsync("Healthy");
    });
});

// Fallback for unmatched routes
app.Run(async context =>
{
    await context.Response.WriteAsync("Default response");
});

app.Run();
```

There is also **`app.MapWhen()`** which branches based on any predicate, not just a path:

```csharp
// Branch based on a custom condition (not just path)
app.MapWhen(
    context => context.Request.Headers.ContainsKey("X-Custom-Header"),
    customApp =>
    {
        customApp.Run(async context =>
        {
            await context.Response.WriteAsync("Custom header detected -- special handling");
        });
    });
```

### Comparison Table

| Method | Receives `next`? | Terminal? | Use Case |
|---|---|---|---|
| `app.Use()` | Yes | No (unless you skip `next()`) | Most middleware: logging, auth checks, header manipulation |
| `app.Run()` | No | Always | Final handler, fallback responses, simple endpoints |
| `app.Map()` | N/A (creates branch) | Creates sub-pipeline | Path-based routing to separate pipelines |
| `app.MapWhen()` | N/A (creates branch) | Creates sub-pipeline | Condition-based branching (any predicate) |
| `app.UseWhen()` | N/A (conditional) | Rejoins main pipeline | Conditionally add middleware but stay in main pipeline |

> [!tip]
> Use `app.UseWhen()` instead of `app.MapWhen()` when you want to conditionally apply middleware but still continue in the main pipeline afterward. `MapWhen` creates a true fork -- the request never returns to the main pipeline. `UseWhen` runs the middleware and then rejoins.

> [!summary] Section Summary
> `app.Use()` is chainable middleware that calls `next()` to continue. `app.Run()` is terminal middleware that ends the pipeline. `app.Map()` branches the pipeline based on path. Choose `Use` for most middleware, `Run` for terminal handlers, and `Map`/`MapWhen`/`UseWhen` for conditional branching.

---

## Inline Middleware with app.Use

The simplest way to write middleware is **inline** using `app.Use()` with a lambda. This is excellent for quick, focused middleware. Here is a complete working example of a real-world scenario -- an order validation middleware:

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Http;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Middleware 1: Request correlation ID
app.Use(async (context, next) =>
{
    // Check if the client sent a correlation ID; generate one if not
    if (!context.Request.Headers.TryGetValue("X-Correlation-Id", out var correlationId))
    {
        correlationId = Guid.NewGuid().ToString();
    }

    // Store it for downstream middleware and services
    context.Items["CorrelationId"] = correlationId.ToString();

    // Add it to the response so the client can trace it
    context.Response.OnStarting(() =>
    {
        context.Response.Headers["X-Correlation-Id"] = correlationId.ToString();
        return Task.CompletedTask;
    });

    await next();
});

// Middleware 2: Request/Response logging
app.Use(async (context, next) =>
{
    var correlationId = context.Items["CorrelationId"] as string;
    var method = context.Request.Method;
    var path = context.Request.Path;

    Console.WriteLine($"[{correlationId}] --> {method} {path}");

    await next();

    Console.WriteLine($"[{correlationId}] <-- {context.Response.StatusCode}");
});

// Middleware 3: Simple API key validation
app.Use(async (context, next) =>
{
    // Skip auth for health check endpoint
    if (context.Request.Path.StartsWithSegments("/health"))
    {
        await next();
        return;
    }

    var apiKey = context.Request.Headers["X-Api-Key"].FirstOrDefault();

    if (string.IsNullOrEmpty(apiKey) || apiKey != "my-secret-key-12345")
    {
        context.Response.StatusCode = StatusCodes.Status401Unauthorized;
        await context.Response.WriteAsJsonAsync(new
        {
            error = "Invalid or missing API key",
            correlationId = context.Items["CorrelationId"]
        });
        return;  // Short-circuit -- do NOT call next()
    }

    await next();
});

// Terminal middleware -- handle requests
app.Map("/api/orders", orderApp =>
{
    orderApp.Run(async context =>
    {
        await context.Response.WriteAsJsonAsync(new
        {
            orders = new[]
            {
                new { id = 1, product = "Widget", quantity = 5 },
                new { id = 2, product = "Gadget", quantity = 3 }
            }
        });
    });
});

app.Map("/health", healthApp =>
{
    healthApp.Run(async context =>
    {
        await context.Response.WriteAsync("OK");
    });
});

app.Run(async context =>
{
    context.Response.StatusCode = 404;
    await context.Response.WriteAsJsonAsync(new { error = "Not found" });
});

app.Run();
```

> [!ad-note]
> Inline middleware is great for small, focused concerns. For more complex middleware with dependencies (injected services, configuration), prefer writing a [[Custom Middleware]] class with the convention-based or factory-based approach.

> [!summary] Section Summary
> Inline middleware uses `app.Use(async (context, next) => { ... })` for quick, lambda-based middleware. It is ideal for simple cross-cutting concerns like correlation IDs, logging, and basic validation. For complex middleware with dependencies, extract it into a dedicated class.

---

## Why Order Matters

The order in which you register middleware in `Program.cs` is the order in which it executes. **Getting the order wrong is one of the most common sources of bugs in ASP.NET Core applications.**

### The Recommended Order

Microsoft recommends this ordering for typical applications:

```csharp
var builder = WebApplication.CreateBuilder(args);
// ... service registration ...
var app = builder.Build();

// 1. Exception/Error handling (outermost -- catches everything)
app.UseExceptionHandler("/error");

// 2. HSTS (HTTP Strict Transport Security)
app.UseHsts();

// 3. HTTPS Redirection
app.UseHttpsRedirection();

// 4. Static Files (serve CSS, JS, images before hitting routing)
app.UseStaticFiles();

// 5. Routing (matches the request to an endpoint)
app.UseRouting();

// 6. CORS (must be after routing, before auth)
app.UseCors();

// 7. Authentication (who are you?)
app.UseAuthentication();

// 8. Authorization (are you allowed?)
app.UseAuthorization();

// 9. Custom middleware
app.Use(async (context, next) => { /* ... */ await next(); });

// 10. Endpoint execution
app.MapControllers();

app.Run();
```

### Concrete Examples of Bugs from Wrong Ordering

**Bug 1: Authorization before Authentication**

```csharp
// WRONG: Authorization runs before the user is identified
app.UseAuthorization();   // Checks permissions -- but User is null!
app.UseAuthentication();  // Identifies the user -- too late

// What happens: Every request to a protected endpoint returns 401 or 403,
// even with valid credentials, because Authorization middleware sees
// an unauthenticated user (context.User has no claims).
```

```csharp
// CORRECT: Authentication first, then Authorization
app.UseAuthentication();  // Identifies the user, populates context.User
app.UseAuthorization();   // Now can check the user's permissions
```

**Bug 2: Static Files after Routing**

```csharp
// WRONG: Static files served after routing
app.UseRouting();
app.UseStaticFiles();     // CSS/JS requests hit the routing pipeline first
app.MapControllers();

// What happens: Requests for /css/site.css go through routing,
// get no match, and return 404 -- even though the file exists.
// Performance also suffers because every static file request
// goes through routing unnecessarily.
```

```csharp
// CORRECT: Static files before routing
app.UseStaticFiles();     // Serves files directly, short-circuits
app.UseRouting();
app.MapControllers();
```

**Bug 3: CORS after Authorization**

```csharp
// WRONG: CORS runs after Authorization
app.UseAuthentication();
app.UseAuthorization();
app.UseCors("AllowFrontend");  // Too late for preflight requests

// What happens: The browser sends an OPTIONS preflight request.
// Authorization middleware rejects it (no auth token on preflight).
// The browser never gets the CORS headers and blocks the real request.
// Your frontend shows: "Access to XMLHttpRequest has been blocked by CORS policy"
```

```csharp
// CORRECT: CORS before Authorization
app.UseAuthentication();
app.UseCors("AllowFrontend");  // Handles preflight before auth checks
app.UseAuthorization();
```

**Bug 4: Exception Handler in the Wrong Position**

```csharp
// WRONG: Exception handler registered too late
app.UseAuthentication();
app.UseAuthorization();
app.UseExceptionHandler("/error");  // Only catches exceptions from middleware below

// What happens: If authentication middleware throws an exception,
// the exception handler never sees it because it is downstream.
// The client receives a raw 500 error with no friendly error page.
```

```csharp
// CORRECT: Exception handler is the outermost middleware
app.UseExceptionHandler("/error");  // Wraps everything -- catches all exceptions
app.UseAuthentication();
app.UseAuthorization();
```

> [!warning] Common Misconception
> Some developers think middleware order only affects performance. In reality, wrong ordering causes **functional bugs** -- authentication failures, CORS errors, missing error pages, and security vulnerabilities. The order is not a suggestion; it is a requirement for correct behavior.

> [!tip]
> When debugging pipeline issues, add a temporary logging middleware at different positions to see what `context.User`, `context.Response.StatusCode`, and `context.Request.Path` look like at each stage. This quickly reveals where things go wrong.

> [!summary] Section Summary
> Middleware order is critical for correct application behavior. Authentication must precede authorization. Static files should come before routing. CORS must handle preflight before authorization rejects it. Exception handling should be the outermost layer to catch all errors. Incorrect ordering causes functional bugs, not just performance issues.

---

## Comparison to ASP.NET HTTP Modules and Handlers

If you have experience with classic ASP.NET (System.Web), it helps to understand how the old model maps to the new middleware model.

| Aspect | ASP.NET (Classic) HTTP Modules/Handlers | ASP.NET Core Middleware |
|---|---|---|
| **Concept** | Separate Modules (cross-cutting) and Handlers (endpoint) | Unified middleware components for both concerns |
| **Configuration** | XML in `web.config` | Code in `Program.cs` / `Startup.cs` |
| **Lifecycle** | Fixed event-based lifecycle (BeginRequest, AuthenticateRequest, etc.) | Flexible delegate chain -- you control the order |
| **Execution model** | Event-driven: modules subscribe to specific pipeline events | Sequential delegate chain: each calls `next()` |
| **Dependency injection** | Not built-in; requires workarounds | First-class DI support via constructor injection |
| **Testability** | Difficult to unit test (tightly coupled to HttpApplication) | Easy to test (pass in mock HttpContext) |
| **Granularity** | Modules are global; Handlers are per-extension/route | Middleware is per-request with full control over branching |
| **Async support** | Limited; async modules added later with complexity | Fully async from the ground up (`Task`-based) |
| **Hosting** | IIS-only | Cross-platform (Kestrel, IIS, Nginx, Docker) |
| **Short-circuiting** | Requires `CompleteRequest()` -- stops events but still runs EndRequest | Simply do not call `next()` -- clean and predictable |

> [!ad-note]
> The classic ASP.NET pipeline had a fixed set of events (BeginRequest, AuthenticateRequest, AuthorizeRequest, etc.) that modules could hook into. ASP.NET Core replaced this rigid system with a simple, composable delegate chain that gives you complete control over the order and behavior of pipeline components.

> [!summary] Section Summary
> The classic ASP.NET model used HTTP Modules (for cross-cutting concerns) and HTTP Handlers (for endpoints) configured via XML and tied to a fixed event lifecycle. ASP.NET Core replaced this with a unified, flexible middleware pipeline that is code-configured, fully async, supports DI natively, and runs cross-platform.

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

---

## Related Topics

- [[Request Pipeline]] -- detailed breakdown of the full ASP.NET Core request pipeline
- [[Custom Middleware]] -- writing middleware as a class with constructor injection
- [[Built-in Middleware]] -- reference for all built-in middleware components (CORS, Auth, Static Files, etc.)
- [[Dependency Injection in ASP.NET Core]] -- how services are resolved in middleware
- [[Exception Handling Middleware]] -- global error handling patterns
- [[Authentication and Authorization]] -- the auth middleware duo in depth
- [[Minimal APIs]] -- how middleware works with the minimal API model

---

## Further Reading

- [[Filters vs Middleware]] -- when to use MVC filters instead of middleware
- [[Endpoint Routing]] -- how routing middleware resolves endpoints
- [[IApplicationBuilder]] -- the interface behind `app.Use()`, `app.Run()`, and `app.Map()`
- [[Middleware Testing]] -- unit testing middleware with `DefaultHttpContext`
- [[Response Caching Middleware]] -- caching responses at the middleware level
- [[Custom Middleware]] -- convention-based and factory-based approaches

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Middleware** is the core mechanism for processing HTTP requests in ASP.NET Core. Each middleware component sits in a pipeline, receives an `HttpContext`, and decides whether to pass the request to the next component via `next()` or short-circuit the pipeline.
>
> **Three fundamental methods** build the pipeline: `app.Use()` for chainable middleware that calls `next()`, `app.Run()` for terminal middleware that ends the pipeline, and `app.Map()` for branching the pipeline by path.
>
> The pipeline follows the **onion model**: code before `await next()` runs on the way in (request phase), and code after runs on the way out (response phase, in reverse order). This means the first registered middleware is the first to see the request and the last to see the response.
>
> **Order is critical** and causes functional bugs when wrong. The standard order is: Exception Handling, HTTPS Redirection, Static Files, Routing, CORS, Authentication, Authorization, Custom Middleware, Endpoints. Authentication must come before Authorization. CORS must handle preflight before Authorization rejects it. Exception handling must be outermost to catch all errors.
>
> The two key types are `RequestDelegate` (the delegate representing each pipeline step) and `HttpContext` (the per-request object carrying all request/response data).
>
> Compared to classic ASP.NET's rigid HTTP Modules/Handlers with XML configuration and fixed event lifecycles, ASP.NET Core middleware is a flexible, code-configured, fully async delegate chain with first-class DI support and cross-platform hosting.
