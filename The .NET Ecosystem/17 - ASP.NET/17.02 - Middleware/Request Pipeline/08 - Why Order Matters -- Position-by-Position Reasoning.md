---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
  - request
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

### Position 2: HSTS (HTTP Strict Transport Security)

**HSTS** tells browsers to only use HTTPS for future requests to this domain. It must come early, before any content is served, so the header is included even in redirect responses.

```csharp
app.UseHsts(); // Adds Strict-Transport-Security header
```

**Why here?** HSTS must be applied before the HTTPS redirect and before any content response. It needs to be inside the exception handler so that if it fails, the error is caught. But it must be before content is served so the browser receives the instruction on every response.

> [!info] Definition
> **HSTS (HTTP Strict Transport Security)** is a security mechanism that tells browsers: "Never contact this domain over plain HTTP again. Always use HTTPS." The browser remembers this for a configurable duration.

### Position 3: HTTPS Redirection

Redirects HTTP requests to HTTPS. This must happen before any content is served over an insecure connection.

```csharp
app.UseHttpsRedirection(); // Redirects http://example.com to https://example.com
```

**Why after HSTS but before StaticFiles?** If HTTPS redirection came after static files, then a request for `http://example.com/styles.css` would serve the CSS file over insecure HTTP instead of redirecting to HTTPS first. All content -- static or dynamic -- must be served securely.

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

### Position 5: Routing

**Routing middleware** examines the request URL and selects which endpoint should handle the request. It does NOT execute the endpoint -- it only selects it.

```csharp
app.UseRouting(); // Selects the matching endpoint
```

**Why here?** Routing must come after static files (so static requests bypass the routing engine) but before authentication and authorization (so the selected endpoint's authorization metadata is available for authorization checks). This is the foundation of the **endpoint routing split**, which is covered in detail below.

### Position 6: CORS

**CORS (Cross-Origin Resource Sharing)** middleware handles preflight OPTIONS requests and adds the appropriate CORS headers to responses.

```csharp
app.UseCors("AllowOrderDashboard"); // Apply named CORS policy
```

**Why between Routing and Authentication?** CORS must come after routing because CORS policies can be endpoint-specific (applied via attributes like `[EnableCors]`). The routing middleware must have already selected the endpoint so CORS knows which policy to apply. CORS must come before authentication because CORS preflight requests (OPTIONS) should never require authentication -- the browser sends them automatically without credentials.

> [!warning] Common Misconception
> Placing CORS after authentication causes preflight OPTIONS requests to fail because the browser does not send authentication tokens with preflight requests. The server returns 401, and the browser blocks the actual request. This is one of the most common CORS configuration bugs.

### Position 7: Authentication

**Authentication middleware** reads credentials from the request (cookies, JWT tokens, etc.) and establishes the user identity (`HttpContext.User`).

```csharp
app.UseAuthentication(); // Identifies who the user is
```

**Why after Routing?** Authentication must come after routing so it can access endpoint metadata. Some endpoints may specify different authentication schemes (e.g., JWT for API endpoints, cookies for web pages). The routing middleware must have selected the endpoint first so authentication knows which scheme to use.

**Why before Authorization?** You must know WHO the user is before you can check WHAT they are allowed to do. Authentication establishes identity; authorization checks permissions against that identity.

### Position 8: Authorization

**Authorization middleware** checks whether the authenticated user has permission to access the selected endpoint.

```csharp
app.UseAuthorization(); // Checks what the user is allowed to do
```

**Why after Authentication?** Authorization depends on the user identity established by authentication. Without authentication running first, `HttpContext.User` would be empty, and all authorization checks would fail.

**Why after Routing?** Authorization needs to know WHICH endpoint was selected so it can check endpoint-specific authorization policies (e.g., `[Authorize(Policy = "AdminOnly")]` on `OrderController`).

> [!danger] Critical Order Dependency
> If you place `UseAuthorization()` before `UseAuthentication()`, authorization will always see an unauthenticated user and deny access to protected endpoints. The application compiles and runs, but every authenticated request fails. This is a silent, hard-to-debug ordering bug.

### Position 9: Custom Middleware

Custom middleware (logging, rate limiting, tenant resolution, request enrichment) goes after the standard framework middleware but before endpoint execution.

```csharp
app.UseMiddleware<RequestLoggingMiddleware>();
app.UseMiddleware<TenantResolutionMiddleware>();
app.UseMiddleware<RateLimitingMiddleware>();
```

**Why here?** At this point, the user is authenticated and authorized, and the endpoint is selected. Custom middleware can safely access the user identity, the selected endpoint metadata, and other context established by prior middleware. Placing custom middleware earlier might mean it runs for requests that will be rejected by authentication or authorization, wasting resources.

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
