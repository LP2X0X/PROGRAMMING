---
tags: [csharp, asp-net-core, middleware, pipeline, request]
aliases: [ASP.NET Request Pipeline, Middleware Pipeline, Request Processing Pipeline, HTTP Request Pipeline]
status: complete
date: 2026-06-18
---

# Request Pipeline

## Overview

The **request pipeline** in ASP.NET Core is the sequence of middleware components that process every HTTP request and response. Each middleware component can inspect, modify, short-circuit, or pass the request to the next component. The order in which middleware is registered in `Program.cs` directly determines how requests flow through the application -- and getting the order wrong is one of the most common sources of subtle bugs in ASP.NET Core applications.

Understanding the pipeline is not optional. It is the backbone of how ASP.NET Core handles cross-cutting concerns like authentication, error handling, logging, and routing.

---

## Table of Contents

- [[#The Pipeline Model]]
- [[#Recommended Middleware Ordering]]
- [[#Why Order Matters -- Position-by-Position Reasoning]]
- [[#The Endpoint Routing Split]]
- [[#Short-Circuiting]]
- [[#Branching the Pipeline]]
- [[#Terminal Middleware]]
- [[#Practical Example -- Full Program.cs]]
- [[#Middleware Order Reference Table]]
- [[#Common Misconceptions]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]
- [[#Further Reading]]

---

## The Pipeline Model

The ASP.NET Core request pipeline is a **chain of delegates** -- each middleware component is a function that receives the `HttpContext` and a reference to the `next` delegate. When a request arrives, it flows through the chain from top to bottom. When a response is generated, it flows back through the chain from bottom to top.

```
Request -->  [Middleware 1]  -->  [Middleware 2]  -->  [Middleware 3]  -->  Endpoint
             <--                 <--                 <--                 Response
```

Each middleware component can:

1. **Do work before** calling `next()` (request processing)
2. **Call `next()`** to pass the request to the next middleware
3. **Do work after** `next()` returns (response processing)
4. **Skip calling `next()`** entirely (short-circuit the pipeline)

```csharp
app.Use(async (context, next) =>
{
    // 1. Work BEFORE the next middleware (request phase)
    var stopwatch = Stopwatch.StartNew();

    await next(context); // 2. Pass to the next middleware

    // 3. Work AFTER the next middleware returns (response phase)
    stopwatch.Stop();
    Console.WriteLine($"Request took {stopwatch.ElapsedMilliseconds}ms");
});
```

> [!summary] Section Summary
> The pipeline is a chain of middleware delegates. Each middleware can process the request, pass it along via `next()`, and optionally process the response on the way back. Skipping `next()` short-circuits the pipeline.

---

## Recommended Middleware Ordering

Microsoft defines a standard ordering for middleware. This is not arbitrary -- each position exists for a specific technical reason. Here is the canonical order:

```csharp
var builder = WebApplication.CreateBuilder(args);

// ... service registration ...

var app = builder.Build();

// 1. Exception Handling (outermost wrapper)
app.UseExceptionHandler("/error");

// 2. HSTS (Strict Transport Security header)
app.UseHsts();

// 3. HTTPS Redirection
app.UseHttpsRedirection();

// 4. Static Files
app.UseStaticFiles();

// 5. Routing (selects endpoint)
app.UseRouting();

// 6. CORS
app.UseCors();

// 7. Authentication (identifies the user)
app.UseAuthentication();

// 8. Authorization (checks permissions)
app.UseAuthorization();

// 9. Custom middleware (your own logic)
app.UseMiddleware<RequestLoggingMiddleware>();

// 10. Endpoints (executes the selected endpoint)
app.MapControllers();
app.MapRazorPages();
```

> [!summary] Section Summary
> The ten-position standard ordering ensures that errors are caught at the outermost layer, security redirects happen before content is served, static files bypass unnecessary processing, routing selects endpoints before authorization checks them, and custom logic runs just before endpoint execution.

---

## Why Order Matters -- Position-by-Position Reasoning

### Position 1: ExceptionHandler / DeveloperExceptionPage

**Exception handling middleware must be first** because it wraps everything else. If any middleware further down the pipeline throws an unhandled exception, this middleware catches it and returns a proper error response instead of a raw 500 error or a connection reset.

```csharp
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage(); // Detailed error pages in dev
}
else
{
    app.UseExceptionHandler("/error"); // User-friendly error page in production
}
```

**Why first?** If exception handling were placed after, say, authentication middleware, then an exception thrown by authentication would crash the application with no graceful error response. By being outermost, it acts as a safety net for the entire pipeline.

> [!warning] Common Misconception
> Some developers put `UseExceptionHandler` after `UseRouting` because they think it only applies to endpoint exceptions. In reality, any middleware can throw -- routing, authentication, custom middleware, and more. Exception handling must wrap all of them.

---

### Position 2: HSTS (HTTP Strict Transport Security)

**HSTS** tells browsers to only use HTTPS for future requests to this domain. It must come early, before any content is served, so the header is included even in redirect responses.

```csharp
app.UseHsts(); // Adds Strict-Transport-Security header
```

**Why here?** HSTS must be applied before the HTTPS redirect and before any content response. It needs to be inside the exception handler so that if it fails, the error is caught. But it must be before content is served so the browser receives the instruction on every response.

> [!info] Definition
> **HSTS (HTTP Strict Transport Security)** is a security mechanism that tells browsers: "Never contact this domain over plain HTTP again. Always use HTTPS." The browser remembers this for a configurable duration.

---

### Position 3: HTTPS Redirection

Redirects HTTP requests to HTTPS. This must happen before any content is served over an insecure connection.

```csharp
app.UseHttpsRedirection(); // Redirects http://example.com to https://example.com
```

**Why after HSTS but before StaticFiles?** If HTTPS redirection came after static files, then a request for `http://example.com/styles.css` would serve the CSS file over insecure HTTP instead of redirecting to HTTPS first. All content -- static or dynamic -- must be served securely.

---

### Position 4: Static Files

**Static file middleware** serves files directly from `wwwroot` (CSS, JavaScript, images, fonts) without any routing, authentication, or authorization processing.

```csharp
app.UseStaticFiles(); // Serves files from wwwroot/
```

**Why before Routing?** Static files do not need routing, authentication, or authorization in most applications. Placing `UseStaticFiles()` before `UseRouting()` means that a request for `/css/site.css` is served immediately and cheaply, without the overhead of running the routing engine, authentication checks, or authorization logic.

> [!tip] Performance Insight
> Static file middleware is a **short-circuiting** middleware. When it finds a matching file, it returns the response and never calls `next()`. This means the routing engine, authentication, and authorization middleware never execute for static file requests. In high-traffic applications, this is a significant performance optimization.

> [!warning] Common Misconception
> Placing `UseStaticFiles()` after `UseAuthorization()` does NOT protect static files with authorization. Static file middleware checks for file existence independent of the authorization system. If you need to protect static files, use a different approach such as serving them through a controller action or using `UseStaticFiles` with `FileServerOptions` that include authorization policies.

---

### Position 5: Routing

**Routing middleware** examines the request URL and selects which endpoint should handle the request. It does NOT execute the endpoint -- it only selects it.

```csharp
app.UseRouting(); // Selects the matching endpoint
```

**Why here?** Routing must come after static files (so static requests bypass the routing engine) but before authentication and authorization (so the selected endpoint's authorization metadata is available for authorization checks). This is the foundation of the **endpoint routing split**, which is covered in detail below.

---

### Position 6: CORS

**CORS (Cross-Origin Resource Sharing)** middleware handles preflight OPTIONS requests and adds the appropriate CORS headers to responses.

```csharp
app.UseCors("AllowOrderDashboard"); // Apply named CORS policy
```

**Why between Routing and Authentication?** CORS must come after routing because CORS policies can be endpoint-specific (applied via attributes like `[EnableCors]`). The routing middleware must have already selected the endpoint so CORS knows which policy to apply. CORS must come before authentication because CORS preflight requests (OPTIONS) should never require authentication -- the browser sends them automatically without credentials.

> [!warning] Common Misconception
> Placing CORS after authentication causes preflight OPTIONS requests to fail because the browser does not send authentication tokens with preflight requests. The server returns 401, and the browser blocks the actual request. This is one of the most common CORS configuration bugs.

---

### Position 7: Authentication

**Authentication middleware** reads credentials from the request (cookies, JWT tokens, etc.) and establishes the user identity (`HttpContext.User`).

```csharp
app.UseAuthentication(); // Identifies who the user is
```

**Why after Routing?** Authentication must come after routing so it can access endpoint metadata. Some endpoints may specify different authentication schemes (e.g., JWT for API endpoints, cookies for web pages). The routing middleware must have selected the endpoint first so authentication knows which scheme to use.

**Why before Authorization?** You must know WHO the user is before you can check WHAT they are allowed to do. Authentication establishes identity; authorization checks permissions against that identity.

---

### Position 8: Authorization

**Authorization middleware** checks whether the authenticated user has permission to access the selected endpoint.

```csharp
app.UseAuthorization(); // Checks what the user is allowed to do
```

**Why after Authentication?** Authorization depends on the user identity established by authentication. Without authentication running first, `HttpContext.User` would be empty, and all authorization checks would fail.

**Why after Routing?** Authorization needs to know WHICH endpoint was selected so it can check endpoint-specific authorization policies (e.g., `[Authorize(Policy = "AdminOnly")]` on `OrderController`).

> [!danger] Critical Order Dependency
> If you place `UseAuthorization()` before `UseAuthentication()`, authorization will always see an unauthenticated user and deny access to protected endpoints. The application compiles and runs, but every authenticated request fails. This is a silent, hard-to-debug ordering bug.

---

### Position 9: Custom Middleware

Custom middleware (logging, rate limiting, tenant resolution, request enrichment) goes after the standard framework middleware but before endpoint execution.

```csharp
app.UseMiddleware<RequestLoggingMiddleware>();
app.UseMiddleware<TenantResolutionMiddleware>();
app.UseMiddleware<RateLimitingMiddleware>();
```

**Why here?** At this point, the user is authenticated and authorized, and the endpoint is selected. Custom middleware can safely access the user identity, the selected endpoint metadata, and other context established by prior middleware. Placing custom middleware earlier might mean it runs for requests that will be rejected by authentication or authorization, wasting resources.

---

### Position 10: Endpoint Middleware (MapControllers, MapRazorPages, etc.)

The **endpoint middleware** executes the selected endpoint -- the controller action, Razor Page handler, minimal API delegate, or SignalR hub.

```csharp
app.MapControllers();           // MVC controllers
app.MapRazorPages();            // Razor Pages
app.MapHub<OrderHub>("/hubs/orders"); // SignalR
app.MapGet("/health", () => Results.Ok("Healthy")); // Minimal API
```

**Why last?** The endpoint is the final destination of the request. All cross-cutting concerns (error handling, security, routing, authentication, authorization, custom logic) must be resolved before the endpoint runs. The endpoint generates the response, which then flows back UP through the pipeline in reverse order.

> [!summary] Section Summary
> Each middleware position exists for a specific technical reason. Exception handling wraps everything. Security headers and redirects happen before content. Static files bypass unnecessary processing. Routing selects endpoints before authentication and authorization check them. Custom middleware runs in a fully resolved context. Endpoints execute last.

---

## The Endpoint Routing Split

This is one of the most critical concepts to understand in ASP.NET Core middleware. **Endpoint routing is split into two phases**:

1. **`UseRouting()`** -- Selects which endpoint matches the request URL
2. **`Map*()`** / endpoint execution -- Executes the selected endpoint

Everything between these two phases has access to the **selected endpoint's metadata** (attributes, authorization policies, CORS policies) but runs BEFORE the endpoint executes.

```
UseRouting()          <-- Selects: "This request matches OrderController.GetOrder()"
    |
    v
UseAuthentication()   <-- Reads JWT token, sets HttpContext.User
    |
    v
UseAuthorization()    <-- Checks [Authorize(Policy = "OrderViewer")] on GetOrder
    |
    v
MapControllers()      <-- Executes OrderController.GetOrder()
```

### Why the Split Matters

The split allows authorization to be **endpoint-aware**. Consider this controller:

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    [HttpGet("{id}")]
    [Authorize(Policy = "OrderViewer")]
    public async Task<IActionResult> GetOrder(int id)
    {
        var order = await _orderService.GetByIdAsync(id);
        return Ok(order);
    }

    [HttpPost]
    [Authorize(Policy = "OrderManager")]
    public async Task<IActionResult> CreateOrder(CreateOrderRequest request)
    {
        var order = await _orderService.CreateAsync(request);
        return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
    }
}
```

When `UseRouting()` runs, it determines that a GET request to `/api/order/42` matches `GetOrder`. This information is stored in `HttpContext`. When `UseAuthorization()` runs next, it reads the endpoint metadata and finds the `[Authorize(Policy = "OrderViewer")]` attribute. It then checks whether the current user satisfies the "OrderViewer" policy.

Without the routing split, authorization would not know which specific action is being called, and therefore could not apply endpoint-specific policies.

### Accessing Endpoint Metadata Between the Split

Any middleware placed between `UseRouting()` and the endpoint can access endpoint metadata:

```csharp
app.UseRouting();

// Custom middleware that reads endpoint metadata
app.Use(async (context, next) =>
{
    var endpoint = context.GetEndpoint();
    if (endpoint != null)
    {
        var auditAttribute = endpoint.Metadata.GetMetadata<AuditLogAttribute>();
        if (auditAttribute != null)
        {
            Console.WriteLine($"Auditing request to: {endpoint.DisplayName}");
        }
    }

    await next(context);
});

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
```

> [!ad-note] Important Implementation Detail
> In modern ASP.NET Core (6+), `UseRouting()` is called implicitly by the framework if you do not call it yourself. However, understanding the split is still critical because the ordering of `UseAuthentication()` and `UseAuthorization()` relative to routing determines whether endpoint metadata is available during those checks.

> [!summary] Section Summary
> Endpoint routing is split into two phases: `UseRouting()` selects the endpoint, and `Map*()` executes it. Middleware between these two phases -- particularly authentication and authorization -- has access to endpoint metadata. This is how ASP.NET Core applies endpoint-specific authorization policies. The split is the reason authorization MUST come after routing.

---

## Short-Circuiting

**Short-circuiting** occurs when a middleware component decides NOT to call `next()`, instead generating a response directly. This stops the request from flowing further down the pipeline.

### How Static File Short-Circuiting Works

The most common example of short-circuiting is the **static file middleware**:

```
Request: GET /css/site.css

[ExceptionHandler] --> [HSTS] --> [HTTPS Redirect] --> [StaticFiles]
                                                            |
                                                      File found!
                                                      Returns CSS
                                                      Does NOT call next()
                                                            |
                                                      <-- Response flows back

[Routing] -- NEVER REACHED
[Authentication] -- NEVER REACHED
[Authorization] -- NEVER REACHED
[MapControllers] -- NEVER REACHED
```

The static file middleware checks if the requested path matches a file in `wwwroot`. If it finds `wwwroot/css/site.css`, it reads the file, sets appropriate content-type headers, writes the file to the response, and returns -- without ever calling `next()`.

### Custom Short-Circuiting Example

You can short-circuit in your own middleware. Here is a **maintenance mode** middleware that returns a 503 for all requests when maintenance is active:

```csharp
public class MaintenanceModeMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IMaintenanceService _maintenanceService;

    public MaintenanceModeMiddleware(RequestDelegate next, IMaintenanceService maintenanceService)
    {
        _next = next;
        _maintenanceService = maintenanceService;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        if (_maintenanceService.IsMaintenanceActive)
        {
            // Short-circuit: do NOT call _next
            context.Response.StatusCode = StatusCodes.Status503ServiceUnavailable;
            context.Response.ContentType = "application/json";
            await context.Response.WriteAsJsonAsync(new
            {
                Error = "Service temporarily unavailable for maintenance",
                EstimatedReturn = _maintenanceService.EstimatedReturnTime
            });
            return; // Pipeline stops here
        }

        await _next(context); // Normal flow continues
    }
}
```

### Another Example: API Key Validation Short-Circuit

```csharp
app.Use(async (context, next) =>
{
    if (context.Request.Path.StartsWithSegments("/api"))
    {
        if (!context.Request.Headers.TryGetValue("X-Api-Key", out var apiKey)
            || apiKey != "expected-key-value")
        {
            context.Response.StatusCode = StatusCodes.Status401Unauthorized;
            await context.Response.WriteAsJsonAsync(new { Error = "Invalid API key" });
            return; // Short-circuit -- no further middleware runs
        }
    }

    await next(context); // Non-API requests pass through
});
```

> [!warning] Common Misconception
> Short-circuiting does not skip the "response phase" of middleware that already ran. If ExceptionHandler, HSTS, and HTTPS Redirect already processed the request before StaticFiles short-circuits, the response still flows back through those three middleware components on the way out. Only middleware AFTER the short-circuiting point is skipped entirely.

> [!summary] Section Summary
> Short-circuiting occurs when middleware returns a response without calling `next()`. Static file middleware is the canonical example -- when it finds a matching file, routing, authentication, and authorization never execute. Custom middleware can short-circuit for scenarios like maintenance mode, API key validation, or IP blocking.

---

## Branching the Pipeline

ASP.NET Core allows you to **branch** the pipeline so that different request paths run through entirely different middleware chains.

### `Map` -- Branch by Path Prefix

`app.Map()` creates a branch that runs only when the request path matches a given prefix:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseExceptionHandler("/error");

// Branch: requests starting with /api get a separate pipeline
app.Map("/api", apiApp =>
{
    apiApp.UseAuthentication();
    apiApp.UseAuthorization();
    apiApp.UseMiddleware<ApiRateLimitingMiddleware>();
    apiApp.UseMiddleware<ApiRequestLoggingMiddleware>();
    apiApp.UseRouting();
    apiApp.MapControllers();
});

// Branch: requests starting with /admin get stricter security
app.Map("/admin", adminApp =>
{
    adminApp.UseAuthentication();
    adminApp.UseAuthorization();
    adminApp.UseMiddleware<AdminAuditMiddleware>();
    adminApp.UseRouting();
    adminApp.MapControllers();
});

// Default pipeline for everything else (public pages)
app.UseStaticFiles();
app.UseRouting();
app.MapRazorPages();

app.Run();
```

> [!ad-note] Path Stripping
> When using `Map("/api", ...)`, the `/api` prefix is **stripped** from the path inside the branch. So a request to `/api/orders` arrives inside the branch as `/orders`. This affects how routing matches controllers and endpoints inside the branch.

### `MapWhen` -- Branch by Condition

`app.MapWhen()` branches based on an arbitrary condition, not just a path prefix:

```csharp
// Branch for requests that accept JSON (API clients)
app.MapWhen(
    context => context.Request.Headers.Accept.ToString().Contains("application/json"),
    jsonApp =>
    {
        jsonApp.UseMiddleware<JsonErrorHandlingMiddleware>();
        jsonApp.UseAuthentication();
        jsonApp.UseAuthorization();
        jsonApp.UseRouting();
        jsonApp.MapControllers();
    }
);

// Branch for requests with a specific tenant header
app.MapWhen(
    context => context.Request.Headers.ContainsKey("X-Tenant-Id"),
    tenantApp =>
    {
        tenantApp.UseMiddleware<TenantResolutionMiddleware>();
        tenantApp.UseAuthentication();
        tenantApp.UseAuthorization();
        tenantApp.UseRouting();
        tenantApp.MapControllers();
    }
);
```

### `UseWhen` -- Conditional Middleware (Rejoins Main Pipeline)

Unlike `Map` and `MapWhen` which create separate pipeline branches, `UseWhen` adds middleware conditionally but **rejoins** the main pipeline afterward:

```csharp
// Add extra logging ONLY for /api requests, then rejoin the main pipeline
app.UseWhen(
    context => context.Request.Path.StartsWithSegments("/api"),
    apiApp =>
    {
        apiApp.UseMiddleware<ApiRequestLoggingMiddleware>();
    }
);

// These still run for ALL requests (including /api)
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

| Method | Branch Type | Rejoins Main Pipeline? | Use Case |
|--------|-------------|----------------------|----------|
| `Map` | Path prefix | No | Separate pipelines for `/api` vs `/admin` |
| `MapWhen` | Arbitrary condition | No | Different handling based on headers, query strings |
| `UseWhen` | Arbitrary condition | Yes | Add extra middleware conditionally without branching |

> [!summary] Section Summary
> `Map` branches by path prefix (strips the prefix), `MapWhen` branches by arbitrary condition, and both create fully separate pipelines. `UseWhen` adds conditional middleware but rejoins the main pipeline. Use branching when different request types need fundamentally different middleware stacks.

---

## Terminal Middleware

**Terminal middleware** is middleware registered with `app.Run()`. It NEVER calls `next()` -- it always ends the pipeline.

```csharp
// Terminal middleware: always responds, never passes to next
app.Run(async context =>
{
    context.Response.ContentType = "text/plain";
    await context.Response.WriteAsync("This is a terminal response. Nothing else will run.");
});
```

### Key Characteristics of `app.Run()`

1. It does not receive a `next` parameter -- there is no option to call the next middleware
2. Any middleware registered AFTER `app.Run()` is never reached
3. It is typically used as a catch-all fallback at the end of the pipeline

### Practical Use: Fallback for Unmatched Requests

```csharp
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.MapRazorPages();

// Terminal fallback: if nothing above handled the request
app.Run(async context =>
{
    context.Response.StatusCode = StatusCodes.Status404NotFound;
    context.Response.ContentType = "application/json";
    await context.Response.WriteAsJsonAsync(new
    {
        Error = "Resource not found",
        Path = context.Request.Path.Value,
        Timestamp = DateTime.UtcNow
    });
});
```

> [!warning] Common Misconception
> `app.Run()` is NOT the same as `app.Use()`. With `app.Use()`, you receive a `next` delegate and CHOOSE whether to call it. With `app.Run()`, there is no `next` parameter at all. It is structurally impossible for terminal middleware to pass the request along.

> [!summary] Section Summary
> Terminal middleware (`app.Run()`) always ends the pipeline. It does not receive a `next` delegate. Use it as a catch-all fallback for requests that no other middleware or endpoint handled.

---

## Practical Example -- Full Program.cs

Here is a complete, production-style `Program.cs` with all middleware in the correct order, annotated with detailed comments:

```csharp
using Microsoft.AspNetCore.HttpOverrides;

var builder = WebApplication.CreateBuilder(args);

// ============================================================
// SERVICE REGISTRATION
// ============================================================
builder.Services.AddControllers();
builder.Services.AddRazorPages();
builder.Services.AddSignalR();

builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer(options =>
    {
        options.Authority = "https://auth.example.com";
        options.Audience = "order-api";
    });

builder.Services.AddAuthorizationBuilder()
    .AddPolicy("OrderViewer", policy => policy.RequireClaim("permission", "orders.read"))
    .AddPolicy("OrderManager", policy => policy.RequireClaim("permission", "orders.write"))
    .AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));

builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowOrderDashboard", policy =>
    {
        policy.WithOrigins("https://dashboard.example.com")
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials();
    });
});

builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<IInventoryService, InventoryService>();

var app = builder.Build();

// ============================================================
// MIDDLEWARE PIPELINE (ORDER MATTERS!)
// ============================================================

// --------------------------------------------------
// 1. EXCEPTION HANDLING (outermost -- catches everything)
// --------------------------------------------------
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/error");
}

// --------------------------------------------------
// 2. HSTS (tell browsers to use HTTPS only)
// --------------------------------------------------
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}

// --------------------------------------------------
// 3. HTTPS REDIRECTION (force secure connections)
// --------------------------------------------------
app.UseHttpsRedirection();

// --------------------------------------------------
// 4. FORWARDED HEADERS (if behind a reverse proxy)
// --------------------------------------------------
app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});

// --------------------------------------------------
// 5. STATIC FILES (short-circuits for wwwroot files)
// --------------------------------------------------
app.UseStaticFiles();

// --------------------------------------------------
// 6. ROUTING (selects the endpoint -- does NOT execute it)
// --------------------------------------------------
app.UseRouting();

// --------------------------------------------------
// 7. CORS (must be after routing, before auth)
// --------------------------------------------------
app.UseCors("AllowOrderDashboard");

// --------------------------------------------------
// 8. AUTHENTICATION (identifies the user)
// --------------------------------------------------
app.UseAuthentication();

// --------------------------------------------------
// 9. AUTHORIZATION (checks permissions for selected endpoint)
// --------------------------------------------------
app.UseAuthorization();

// --------------------------------------------------
// 10. CUSTOM MIDDLEWARE (runs in authenticated context)
// --------------------------------------------------
app.UseMiddleware<RequestLoggingMiddleware>();
app.UseMiddleware<TenantResolutionMiddleware>();

// --------------------------------------------------
// 11. ENDPOINTS (executes the selected endpoint)
// --------------------------------------------------
app.MapControllers();
app.MapRazorPages();
app.MapHub<OrderNotificationHub>("/hubs/orders");

// Health check endpoint (no auth required)
app.MapGet("/health", () => Results.Ok(new { Status = "Healthy", Timestamp = DateTime.UtcNow }))
   .AllowAnonymous();

// --------------------------------------------------
// 12. TERMINAL FALLBACK (catches unmatched requests)
// --------------------------------------------------
app.Run(async context =>
{
    context.Response.StatusCode = StatusCodes.Status404NotFound;
    context.Response.ContentType = "application/json";
    await context.Response.WriteAsJsonAsync(new
    {
        Error = "The requested resource was not found",
        Path = context.Request.Path.Value
    });
});

app.Run();
```

> [!tip] Practical Tip
> In production, you often also need `UseForwardedHeaders()` when running behind a reverse proxy like Nginx or a load balancer. Place it after HTTPS redirection but before static files. Without it, the application may see internal IP addresses and HTTP scheme instead of the client's real IP and HTTPS.

> [!summary] Section Summary
> A production `Program.cs` follows the ten-position ordering strictly. Exception handling wraps everything, security middleware runs early, static files short-circuit, routing selects endpoints, CORS and auth run in the routing split, custom middleware runs in authenticated context, and endpoints execute last.

---

## Middleware Order Reference Table

| Position | Middleware | What It Does | Why This Position |
|----------|-----------|-------------|-------------------|
| 1 | `UseExceptionHandler()` / `UseDeveloperExceptionPage()` | Catches unhandled exceptions, returns error responses | Must wrap ALL other middleware to catch any exception from any component |
| 2 | `UseHsts()` | Adds `Strict-Transport-Security` header | Must run before content is served so the header appears on all responses |
| 3 | `UseHttpsRedirection()` | Redirects HTTP to HTTPS | Must run before any content (static or dynamic) is served over insecure HTTP |
| 4 | `UseStaticFiles()` | Serves CSS, JS, images from `wwwroot` | Before routing so static requests bypass the routing engine, auth, and other overhead |
| 5 | `UseRouting()` | Selects which endpoint matches the request URL | Before auth so that auth can read endpoint metadata (policies, attributes) |
| 6 | `UseCors()` | Handles CORS preflight and headers | After routing (needs endpoint metadata) but before auth (preflight has no credentials) |
| 7 | `UseAuthentication()` | Reads credentials, establishes `HttpContext.User` | After routing (may need endpoint-specific auth schemes), before authorization |
| 8 | `UseAuthorization()` | Checks user permissions against endpoint policies | After authentication (needs user identity) and after routing (needs endpoint metadata) |
| 9 | Custom middleware | Your application-specific logic | After all framework middleware so full context (user, endpoint, etc.) is available |
| 10 | `MapControllers()` / `MapRazorPages()` / `Map*()` | Executes the selected endpoint | Last -- all cross-cutting concerns must be resolved before the endpoint runs |

> [!summary] Section Summary
> The reference table provides a quick lookup for the correct position of each middleware and the reasoning behind it. Each position depends on the middleware before and after it.

---

## Common Misconceptions

> [!warning] Common Misconception
> **"Middleware order does not matter because ASP.NET Core figures it out."** This is false. ASP.NET Core executes middleware in exactly the order you register it. There is no automatic reordering. If you put authorization before authentication, authorization will always see an unauthenticated user.

> [!warning] Common Misconception
> **"UseStaticFiles after UseAuthorization protects my static files."** Static file middleware does not participate in the authorization system. It checks for file existence on disk and serves the file regardless of authorization status. To protect static files, serve them through a controller action or use `MapStaticAssets()` with authorization.

> [!warning] Common Misconception
> **"I can call UseRouting() and UseEndpoints() multiple times to create multiple pipelines."** While technically possible, each call to `UseRouting()` starts a new routing scope. This is rarely needed and usually indicates a misunderstanding. Use `Map()` branching instead for separate pipelines.

> [!warning] Common Misconception
> **"UseRouting() is not needed because MapControllers() handles routing."** In modern ASP.NET Core (6+), `UseRouting()` is called implicitly. However, if you need middleware to access endpoint metadata BEFORE authentication runs (such as a custom CORS handler), you must call `UseRouting()` explicitly to control where the routing phase occurs in the pipeline.

> [!summary] Section Summary
> The most dangerous misconceptions involve assuming middleware ordering is automatic, believing static files are protected by authorization middleware, and not understanding that `UseRouting()` is implicitly called but can be made explicit when precise ordering control is needed.

---

## Comprehensive Summary

> [!tip] Complete Summary
> The ASP.NET Core request pipeline is a sequential chain of middleware components that process every HTTP request. The standard ordering is: (1) Exception Handling, (2) HSTS, (3) HTTPS Redirection, (4) Static Files, (5) Routing, (6) CORS, (7) Authentication, (8) Authorization, (9) Custom Middleware, (10) Endpoints.
>
> Each position exists for a specific reason. Exception handling wraps everything to catch errors from any component. HSTS and HTTPS redirection enforce security before any content is served. Static files short-circuit the pipeline to avoid unnecessary routing and auth overhead. Routing selects the endpoint so that CORS, authentication, and authorization can access endpoint metadata. Authorization depends on both routing (to know which endpoint) and authentication (to know who the user is). Custom middleware runs in a fully resolved context with access to user identity and endpoint information. Endpoints execute last.
>
> The **endpoint routing split** is the critical architectural concept: `UseRouting()` selects the endpoint, and middleware between routing and endpoint execution can read endpoint metadata. This is how ASP.NET Core applies endpoint-specific authorization policies.
>
> **Short-circuiting** allows middleware to return a response without passing the request further. Static files short-circuit for performance; custom middleware can short-circuit for maintenance mode, rate limiting, or validation failures.
>
> **Pipeline branching** via `Map()`, `MapWhen()`, and `UseWhen()` creates separate middleware chains for different request types. **Terminal middleware** via `app.Run()` always ends the pipeline and is used as a catch-all fallback.

---

## Related Topics

- [[Middleware Overview]]
- [[Custom Middleware]]
- [[Built-in Middleware]]
- [[Authentication and Authorization]]
- [[Routing in ASP.NET Core]]
- [[Static Files Configuration]]
- [[CORS Configuration]]
- [[Error Handling Middleware]]
- [[Endpoint Routing]]
- [[Minimal APIs]]

---

## Further Reading

- [[Dependency Injection in ASP.NET Core]]
- [[Program.cs and Host Builder]]
- [[Kestrel Web Server]]
- [[Reverse Proxy Configuration]]
- [[Security Headers]]
- [[Rate Limiting Middleware]]
- [[Health Checks]]
- [[SignalR Hubs]]
