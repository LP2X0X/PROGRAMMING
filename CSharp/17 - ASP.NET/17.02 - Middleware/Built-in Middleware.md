---
tags: [csharp, asp-net-core, middleware, built-in]
aliases: [Built-in Middleware Components, ASP.NET Core Built-in Middleware, Default Middleware]
status: complete
date: 2026-06-18
---

# Built-in Middleware

ASP.NET Core ships with a rich set of **built-in middleware** components that handle cross-cutting concerns such as error handling, security, routing, caching, and compression. Each middleware is a discrete component in the [[Request Pipeline]] that processes HTTP requests and responses in a specific order. Understanding what each built-in middleware does, where it belongs in the pipeline, and how to configure it is essential for building robust web applications.

This note provides a complete reference for the most important built-in middleware components. For foundational concepts, see [[Middleware Overview]]. For writing your own components, see [[Custom Middleware]].

---

## Table of Contents

- [[#Quick Reference Table]]
- [[#Recommended Pipeline Order]]
- [[#UseExceptionHandler]]
- [[#UseDeveloperExceptionPage]]
- [[#UseHsts]]
- [[#UseHttpsRedirection]]
- [[#UseStaticFiles]]
- [[#UseRouting]]
- [[#UseCors]]
- [[#UseAuthentication]]
- [[#UseAuthorization]]
- [[#UseSession]]
- [[#UseResponseCaching]]
- [[#UseResponseCompression]]
- [[#UseRateLimiter]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]
- [[#Further Reading]]

---

## Quick Reference Table

| Middleware | Purpose | Required? | Typical Position | Key Gotcha |
|---|---|---|---|---|
| `UseExceptionHandler` | Catches unhandled exceptions and returns error page/JSON | Yes (production) | Very early (first or second) | Must be before all middleware that can throw |
| `UseDeveloperExceptionPage` | Shows detailed stack trace page for developers | No (dev only) | Very first | **Never** enable in production -- exposes internals |
| `UseHsts` | Adds `Strict-Transport-Security` header | Recommended | Before HTTPS redirect | Do not use in development -- browsers cache the header |
| `UseHttpsRedirection` | Redirects HTTP requests to HTTPS | Recommended | After HSTS, before static files | Fails silently if HTTPS port is unknown |
| `UseStaticFiles` | Serves files from `wwwroot` or custom paths | If serving static content | Before routing | Does not serve files outside configured paths by default |
| `UseRouting` | Matches request URL to an endpoint | Yes | After static files, before auth | Must come before `UseAuthorization` |
| `UseCors` | Adds CORS headers for cross-origin requests | If API consumed by browsers | Between routing and auth | Order matters -- must be after `UseRouting`, before `UseAuthorization` |
| `UseAuthentication` | Runs authentication handler, sets `HttpContext.User` | If using auth | After routing, before authorization | Does not reject unauthenticated requests on its own |
| `UseAuthorization` | Enforces `[Authorize]` policies | If using auth | After authentication | Requires `UseAuthentication` before it |
| `UseSession` | Enables session state via cookies | If using sessions | After CORS, before endpoints | Default session cookie is not essential -- GDPR implications |
| `UseResponseCaching` | Caches responses based on HTTP cache headers | Optional performance | After CORS, before endpoints | Does not cache if `Authorization` header is present |
| `UseResponseCompression` | Compresses response body with gzip/brotli | Optional performance | Very early (before static files) | Do not use with HTTPS in security-sensitive scenarios (BREACH attack) |
| `UseRateLimiter` | Limits request rate per client/endpoint | Optional (.NET 7+) | After routing, before auth or endpoints | Requires `.AddRateLimiter()` in service registration |

> [!summary] Section Summary
> The reference table above gives a bird's-eye view of each built-in middleware, its purpose, whether it is required, where it typically sits in the pipeline, and the single most important gotcha to watch for.

---

## Recommended Pipeline Order

The order in which you register middleware determines the order they execute. ASP.NET Core does not enforce order -- it is your responsibility. The following is the **recommended order** from the official documentation:

```csharp
var app = builder.Build();

// 1. Exception handling (outermost layer)
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

// 2. HTTPS redirection
app.UseHttpsRedirection();

// 3. Static files (short-circuit before routing)
app.UseStaticFiles();

// 4. Routing (matches URL to endpoint)
app.UseRouting();

// 5. CORS (must be between routing and authorization)
app.UseCors();

// 6. Authentication and Authorization
app.UseAuthentication();
app.UseAuthorization();

// 7. Session
app.UseSession();

// 8. Response caching / compression
app.UseResponseCaching();

// 9. Rate limiting
app.UseRateLimiter();

// 10. Endpoint execution (controllers, minimal APIs, Razor Pages)
app.MapControllers();
app.MapRazorPages();
```

> [!warning]
> Placing middleware out of order is one of the most common sources of subtle bugs in ASP.NET Core applications. For example, placing `UseAuthorization` before `UseRouting` will cause authorization to fail because no endpoint has been matched yet.

> [!summary] Section Summary
> The recommended pipeline order ensures that each middleware has the context it needs from earlier middleware. Exception handling wraps everything, HTTPS and static files come before routing, and authentication must precede authorization.

---

## UseExceptionHandler

**`UseExceptionHandler`** is the production-grade exception handling middleware. It catches any unhandled exception thrown by downstream middleware and generates an appropriate error response without exposing sensitive details to the client.

### How It Works

When an exception propagates up the pipeline, this middleware:
1. Catches the exception
2. Logs it
3. Clears the response
4. Re-executes the pipeline using a specified error-handling path (e.g., `/Error`)
5. The error-handling endpoint can inspect `IExceptionHandlerPathFeature` to access the original exception

### Configuration for MVC (HTML Error Page)

```csharp
// Program.cs
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
}
```

```csharp
// HomeController.cs
[AllowAnonymous]
[ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
public IActionResult Error()
{
    var exceptionFeature = HttpContext.Features.Get<IExceptionHandlerPathFeature>();

    // Log the exception
    _logger.LogError(exceptionFeature?.Error, 
        "Unhandled exception at path: {Path}", 
        exceptionFeature?.Path);

    return View(new ErrorViewModel
    {
        RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier,
        Message = "An unexpected error occurred. Please try again later."
    });
}
```

### Configuration for Web API (JSON Error Response)

```csharp
// Program.cs
app.UseExceptionHandler(appError =>
{
    appError.Run(async context =>
    {
        context.Response.StatusCode = StatusCodes.Status500InternalServerError;
        context.Response.ContentType = "application/json";

        var exceptionFeature = context.Features.Get<IExceptionHandlerPathFeature>();
        var exception = exceptionFeature?.Error;

        var logger = context.RequestServices.GetRequiredService<ILogger<Program>>();
        logger.LogError(exception, "Unhandled exception in request pipeline");

        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status500InternalServerError,
            Title = "Internal Server Error",
            Detail = app.Environment.IsDevelopment() 
                ? exception?.Message 
                : "An error occurred processing your request."
        };

        await context.Response.WriteAsJsonAsync(problemDetails);
    });
});
```

### .NET 8+ `IExceptionHandler` Interface

Starting with .NET 8, you can register custom exception handlers via dependency injection:

```csharp
// OrderExceptionHandler.cs
public class OrderExceptionHandler : IExceptionHandler
{
    private readonly ILogger<OrderExceptionHandler> _logger;

    public OrderExceptionHandler(ILogger<OrderExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception, "Exception while processing order request");

        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status500InternalServerError,
            Title = "Server Error",
            Type = "https://tools.ietf.org/html/rfc7231#section-6.6.1"
        };

        httpContext.Response.StatusCode = StatusCodes.Status500InternalServerError;
        await httpContext.Response.WriteAsJsonAsync(problemDetails, cancellationToken);

        return true; // true = exception was handled
    }
}

// Program.cs
builder.Services.AddExceptionHandler<OrderExceptionHandler>();
app.UseExceptionHandler();
```

### When You Need It

Always in production. Every ASP.NET Core application should have `UseExceptionHandler` registered for non-development environments.

### Gotchas

- Must be placed **very early** in the pipeline so it catches exceptions from all downstream middleware
- The re-execution path (`/Error`) runs through the entire pipeline again, so ensure the error endpoint cannot itself throw
- Response headers that were already sent before the exception cannot be modified -- the middleware clears the response only if headers have not yet been flushed

> [!summary] Section Summary
> `UseExceptionHandler` is the backbone of production error handling. For MVC apps, it redirects to an error page. For APIs, it returns structured JSON (ideally `ProblemDetails`). In .NET 8+, the `IExceptionHandler` interface offers a cleaner DI-friendly approach. Always place it at the outermost layer of the pipeline.

---

## UseDeveloperExceptionPage

**`UseDeveloperExceptionPage`** renders a detailed, interactive error page when an unhandled exception occurs. It shows the full stack trace, source code snippets around the exception, request headers, cookies, query strings, and routing data.

### Configuration

```csharp
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
```

> [!danger]
> **Never enable `UseDeveloperExceptionPage` in production.** It exposes internal implementation details including:
> - Full stack traces with file paths and line numbers
> - Source code snippets
> - Environment variable names
> - Connection strings visible in exception messages
> - Internal class and method names
>
> This information is invaluable for attackers performing reconnaissance on your application.

### What the Page Shows

- **Exception details**: type, message, and full stack trace
- **Source code**: the exact lines around where the exception was thrown (if PDB files are available)
- **Query string**: all query parameters
- **Cookies**: all cookies sent with the request
- **Headers**: all HTTP request headers
- **Routing**: matched route data

### Automatic in .NET 6+

In .NET 6 and later, the developer exception page is automatically enabled when the application runs in the `Development` environment. You do not need to explicitly call `UseDeveloperExceptionPage()` if you are using `WebApplication.CreateBuilder()` -- it is added by default. However, explicit registration makes the intent clear.

### When You Need It

Only during local development and debugging.

### Gotchas

- If both `UseDeveloperExceptionPage` and `UseExceptionHandler` are registered, the one registered first wins because it catches the exception before the other sees it
- The developer exception page **does not log exceptions** -- it only displays them. You still need logging middleware or `ILogger` in production

> [!summary] Section Summary
> `UseDeveloperExceptionPage` is a powerful debugging aid that shows comprehensive error details. It is strictly for development use and is automatically enabled in .NET 6+ development environments. Never deploy it to production.

---

## UseHsts

**`UseHsts`** adds the `Strict-Transport-Security` HTTP response header, which instructs browsers to only access the site over HTTPS for a specified duration. This is called **HTTP Strict Transport Security (HSTS)**.

### How HSTS Works

When a browser receives the `Strict-Transport-Security` header, it:
1. Remembers that this domain requires HTTPS
2. Automatically upgrades all future HTTP requests to HTTPS before sending them
3. Refuses to connect over plain HTTP, even if the user types `http://` in the address bar
4. Prevents users from clicking through certificate warnings for this domain

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddHsts(options =>
{
    options.Preload = true;
    options.IncludeSubDomains = true;
    options.MaxAge = TimeSpan.FromDays(365);
    options.ExcludedHosts.Add("staging.orderportal.com");
});

// Program.cs -- middleware
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}

app.UseHttpsRedirection();
```

### Configuration Options

| Option | Default | Description |
|---|---|---|
| `MaxAge` | 30 days | How long browsers remember to use HTTPS |
| `IncludeSubDomains` | `false` | Apply HSTS to all subdomains |
| `Preload` | `false` | Allow inclusion in browser HSTS preload lists |
| `ExcludedHosts` | `localhost`, etc. | Hosts that should not receive the header |

### When You Need It

Any production application served over HTTPS (which should be every application).

### Gotchas

- **Do not use in development** -- browsers cache the HSTS header, and it can make `localhost` inaccessible over HTTP for the duration of `MaxAge`
- Setting `Preload = true` alone does not preload your site -- you must also submit your domain to the browser preload list at `hstspreload.org`
- Start with a short `MaxAge` (e.g., 1 hour) and gradually increase once you confirm HTTPS works correctly across your entire site
- HSTS only takes effect after the **first successful HTTPS response** -- the very first request is still vulnerable to a man-in-the-middle attack (unless preloaded)

> [!summary] Section Summary
> `UseHsts` tells browsers to always use HTTPS for your domain. Configure `MaxAge`, `IncludeSubDomains`, and `Preload` carefully. Only enable it in non-development environments because browsers cache the directive aggressively.

---

## UseHttpsRedirection

**`UseHttpsRedirection`** automatically redirects HTTP requests to HTTPS by issuing a `307 Temporary Redirect` (or `301 Permanent Redirect`) response.

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddHttpsRedirection(options =>
{
    options.RedirectStatusCode = StatusCodes.Status301MovedPermanently;
    options.HttpsPort = 5001;
});

// Program.cs -- middleware
app.UseHttpsRedirection();
```

### How the HTTPS Port is Determined

The middleware determines the HTTPS port in this order:
1. `HttpsRedirectionOptions.HttpsPort` (explicit configuration)
2. `ASPNETCORE_HTTPS_PORT` environment variable
3. `IServerAddressesFeature` (inspects the server's bound addresses)

If none of these provide a port, the middleware **logs a warning and does not redirect**. This is a common source of confusion in containerized environments.

### When You Need It

Any application that accepts both HTTP and HTTPS traffic and wants to enforce HTTPS.

### Gotchas

- In Docker/Kubernetes environments, the application often sits behind a reverse proxy that terminates TLS. In this case, HTTPS redirection can cause redirect loops. Use `UseForwardedHeaders` before `UseHttpsRedirection` to respect `X-Forwarded-Proto`
- If the HTTPS port cannot be determined, the middleware silently does nothing -- always verify it is working
- Use `307` (temporary) during development and testing; switch to `301` (permanent) only when you are confident HTTPS is fully configured

> [!ad-note]
> When behind a load balancer or reverse proxy, configure forwarded headers:
> ```csharp
> builder.Services.Configure<ForwardedHeadersOptions>(options =>
> {
>     options.ForwardedHeaders = ForwardedHeaders.XForwardedFor 
>         | ForwardedHeaders.XForwardedProto;
> });
> 
> app.UseForwardedHeaders();
> app.UseHttpsRedirection();
> ```

> [!summary] Section Summary
> `UseHttpsRedirection` enforces HTTPS by redirecting HTTP traffic. The HTTPS port must be discoverable or the middleware silently becomes a no-op. Be aware of reverse proxy scenarios that require `UseForwardedHeaders`.

---

## UseStaticFiles

**`UseStaticFiles`** serves static files (HTML, CSS, JavaScript, images, etc.) directly from the `wwwroot` directory without passing through the full middleware pipeline. It acts as a **short-circuit** middleware -- if a matching file is found, it serves it immediately and does not call the next middleware.

### Basic Configuration

```csharp
// Serves files from wwwroot/
app.UseStaticFiles();
```

### Serving from a Custom Directory

```csharp
// Serve files from a custom directory at a custom URL path
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "ProductImages")),
    RequestPath = "/images"
});
```

This maps requests like `/images/product-001.jpg` to the file `ProductImages/product-001.jpg` on disk.

### Setting Cache Headers

```csharp
app.UseStaticFiles(new StaticFileOptions
{
    OnPrepareResponse = ctx =>
    {
        // Cache static files for 30 days
        ctx.Context.Response.Headers.Append(
            "Cache-Control", "public,max-age=2592000");
    }
});
```

### Serving Files with Custom Content Types

```csharp
var contentTypeProvider = new FileExtensionContentTypeProvider();
contentTypeProvider.Mappings[".webmanifest"] = "application/manifest+json";
contentTypeProvider.Mappings[".data"] = "application/octet-stream";

app.UseStaticFiles(new StaticFileOptions
{
    ContentTypeProvider = contentTypeProvider
});
```

### When You Need It

Any application that serves static assets (CSS, JS, images, fonts). Placed **before** `UseRouting` so static file requests are handled quickly without routing overhead.

### Gotchas

- By default, only files in `wwwroot` are served. Files outside this directory are not accessible unless you explicitly configure a `PhysicalFileProvider`
- `UseStaticFiles` does **not** enable directory browsing by default. Use `UseDirectoryBrowser` separately if needed (not recommended in production)
- Files without a recognized MIME type are not served. Use `ServeUnknownFileTypes = true` with caution -- it can expose unintended files
- Place `UseStaticFiles` **before** `UseRouting` and `UseAuthorization` so that static file requests do not incur authentication/authorization overhead

> [!summary] Section Summary
> `UseStaticFiles` serves files from `wwwroot` (or custom paths) as a short-circuit middleware. Configure cache headers for performance, use `PhysicalFileProvider` for custom directories, and always place it before routing to avoid unnecessary processing.

---

## UseRouting

**`UseRouting`** is responsible for matching incoming HTTP requests to **endpoints** defined in your application (controllers, minimal APIs, Razor Pages, SignalR hubs, gRPC services, etc.). It works in tandem with the **endpoint middleware** that actually executes the matched endpoint.

### How Routing Works

The routing system operates in two phases:
1. **`UseRouting`** -- examines the request URL and selects the best matching endpoint. It stores the matched endpoint in `HttpContext` via `IEndpointFeature`
2. **Endpoint execution** (e.g., `MapControllers`, `MapGet`) -- runs the selected endpoint's delegate

Any middleware placed **between** `UseRouting` and the endpoint execution can inspect the matched endpoint before it runs. This is how `UseAuthorization` knows which `[Authorize]` attributes to enforce.

### Configuration

```csharp
app.UseRouting();

// Middleware here can see which endpoint was matched
app.UseAuthentication();
app.UseAuthorization();

// Endpoints are executed here
app.MapControllers();
app.MapRazorPages();
```

### Inspecting the Matched Endpoint

```csharp
// Custom middleware that checks the matched endpoint
app.Use(async (context, next) =>
{
    var endpoint = context.GetEndpoint();
    if (endpoint != null)
    {
        var metadata = endpoint.Metadata.GetMetadata<IAuthorizeData>();
        // Log or inspect authorization requirements
    }
    await next(context);
});
```

### Implicit Routing in .NET 6+

In .NET 6 and later, `UseRouting` is implicitly called by `WebApplication` if you do not call it explicitly. However, calling it explicitly is necessary when you need to place middleware between routing and endpoint execution (which is common).

### When You Need It

Every ASP.NET Core application that uses endpoint routing (which is all modern ASP.NET Core apps).

### Gotchas

- `UseRouting` must come **before** `UseAuthorization` -- authorization needs to know the matched endpoint to evaluate `[Authorize]` attributes
- `UseRouting` must come **before** `UseCors` for CORS to apply policies based on endpoint metadata
- In .NET 6+, if you do not call `UseRouting` explicitly, the framework inserts it automatically at the beginning of the pipeline, which means middleware like `UseAuthentication` might not be able to see the matched endpoint

> [!summary] Section Summary
> `UseRouting` matches URLs to endpoints and makes the matched endpoint available to downstream middleware. It pairs with endpoint execution middleware (`MapControllers`, etc.). Explicit placement is important when other middleware (auth, CORS) needs to inspect endpoint metadata.

---

## UseCors

**`UseCors`** adds **Cross-Origin Resource Sharing** headers to responses, allowing browsers to make requests from a different origin (domain, scheme, or port) than the one serving the application.

### Why CORS Exists

Browsers enforce the **Same-Origin Policy**, which blocks JavaScript from making requests to a different origin. CORS is the mechanism that relaxes this restriction in a controlled way. Without it, your API cannot be consumed by a frontend hosted on a different domain.

### Named Policy Configuration

```csharp
// Program.cs -- services
builder.Services.AddCors(options =>
{
    options.AddPolicy("OrderPortalPolicy", policy =>
    {
        policy.WithOrigins(
                "https://orderportal.example.com",
                "https://admin.example.com")
            .WithMethods("GET", "POST", "PUT", "DELETE")
            .WithHeaders("Content-Type", "Authorization")
            .AllowCredentials()
            .SetPreflightMaxAge(TimeSpan.FromMinutes(10));
    });

    options.AddPolicy("PublicApiPolicy", policy =>
    {
        policy.AllowAnyOrigin()
            .AllowAnyMethod()
            .AllowAnyHeader();
    });
});

// Program.cs -- middleware
app.UseRouting();
app.UseCors("OrderPortalPolicy"); // default policy

// Or apply per-endpoint:
app.MapControllers();
```

### Default Policy Configuration

```csharp
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
    {
        policy.WithOrigins("https://orderportal.example.com")
            .AllowAnyMethod()
            .AllowAnyHeader();
    });
});

// No policy name needed when using default
app.UseCors();
```

### Per-Endpoint CORS

```csharp
app.MapGet("/api/products", () => Results.Ok(products))
    .RequireCors("PublicApiPolicy");

// Or via attribute on controllers:
[EnableCors("OrderPortalPolicy")]
public class OrdersController : ControllerBase { }
```

### When You Need It

Whenever your API is consumed by a browser-based client hosted on a different origin.

### Gotchas

- `UseCors` must be placed **after** `UseRouting` and **before** `UseAuthorization` so that CORS preflight requests are handled before authorization rejects them
- `AllowAnyOrigin()` and `AllowCredentials()` **cannot be combined** -- the CORS specification forbids it. This will throw a runtime exception
- CORS is a **browser-enforced** security mechanism. Non-browser clients (Postman, `HttpClient`) ignore CORS entirely
- Forgetting to include `Authorization` in `WithHeaders` when your API requires bearer tokens causes preflight failures

> [!summary] Section Summary
> `UseCors` controls which origins can access your API from a browser. Use named policies for fine-grained control or a default policy for simplicity. Place it between `UseRouting` and `UseAuthorization`, and never combine `AllowAnyOrigin` with `AllowCredentials`.

---

## UseAuthentication

**`UseAuthentication`** runs the configured **authentication handler** for each request. It examines the request (typically looking for a cookie, bearer token, or other credential) and populates `HttpContext.User` with a `ClaimsPrincipal` representing the authenticated identity.

### What Happens Internally

1. The middleware calls `IAuthenticationService.AuthenticateAsync()` using the **default authentication scheme**
2. The authentication handler (e.g., cookie handler, JWT bearer handler) inspects the request for credentials
3. If valid credentials are found, it creates a `ClaimsPrincipal` and assigns it to `HttpContext.User`
4. If no valid credentials are found, `HttpContext.User` is set to an anonymous/empty principal
5. The middleware **does not reject the request** -- it only identifies who the caller is

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://auth.example.com";
        options.Audience = "order-api";
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            ValidateIssuer = true,
            ValidateAudience = true,
            ClockSkew = TimeSpan.FromMinutes(1)
        };
    });

// Program.cs -- middleware
app.UseAuthentication();
app.UseAuthorization();
```

### Multiple Authentication Schemes

```csharp
builder.Services.AddAuthentication()
    .AddJwtBearer("Bearer", options => { /* JWT config */ })
    .AddCookie("Cookies", options => { /* Cookie config */ });

// On a controller, specify which scheme:
[Authorize(AuthenticationSchemes = "Bearer")]
public class ApiOrdersController : ControllerBase { }

[Authorize(AuthenticationSchemes = "Cookies")]
public class WebOrdersController : Controller { }
```

### When You Need It

Any application that needs to know who the caller is, regardless of whether authorization is enforced.

### Gotchas

- `UseAuthentication` **does not deny access** to unauthenticated users -- that is the job of `UseAuthorization`
- Must be placed **before** `UseAuthorization` and **after** `UseRouting`
- If you forget to call `UseAuthentication`, `[Authorize]` attributes will still block requests, but `HttpContext.User` will never be populated -- leading to confusing 401 responses even with valid tokens

> [!summary] Section Summary
> `UseAuthentication` identifies the caller by running the configured authentication handler and populating `HttpContext.User`. It does not enforce access control -- that responsibility belongs to `UseAuthorization`. Always place it between `UseRouting` and `UseAuthorization`.

---

## UseAuthorization

**`UseAuthorization`** evaluates authorization policies and attributes (`[Authorize]`, `[AllowAnonymous]`) against the authenticated user. It is the enforcement point that determines whether a request is allowed to proceed to the endpoint.

### How It Relates to Authentication

| Concern | Middleware | Question Answered |
|---|---|---|
| Authentication | `UseAuthentication` | "Who are you?" |
| Authorization | `UseAuthorization` | "Are you allowed to do this?" |

Authentication must run **before** authorization. If the user is not authenticated and the endpoint requires authorization, the middleware triggers an **authentication challenge** (e.g., redirect to login page or 401 response).

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));

    options.AddPolicy("CanManageOrders", policy =>
        policy.RequireClaim("permission", "orders.manage"));

    // Fallback policy: require authenticated user for all endpoints by default
    options.FallbackPolicy = new AuthorizationPolicyBuilder()
        .RequireAuthenticatedUser()
        .Build();
});

// Program.cs -- middleware
app.UseAuthentication();
app.UseAuthorization();
```

### Applying Policies

```csharp
[Authorize(Policy = "CanManageOrders")]
public class OrdersController : ControllerBase
{
    [AllowAnonymous]
    public IActionResult GetPublicCatalog() => Ok();

    public IActionResult CreateOrder(OrderRequest request) => Ok();
}

// Minimal API
app.MapDelete("/api/orders/{id}", (int id) => Results.NoContent())
    .RequireAuthorization("AdminOnly");
```

### When You Need It

Any application that needs to restrict access to endpoints based on user identity, roles, or claims.

### Gotchas

- `UseAuthorization` **must** come after `UseRouting` because it needs to know which endpoint was matched (and its metadata)
- `UseAuthorization` **must** come after `UseAuthentication` -- otherwise `HttpContext.User` is not yet populated
- Setting a `FallbackPolicy` that requires authentication means **every** endpoint requires authentication unless explicitly marked `[AllowAnonymous]`. This includes health checks and static files served after the authorization middleware
- `[AllowAnonymous]` on a controller method **overrides** `[Authorize]` on the controller class

> [!summary] Section Summary
> `UseAuthorization` enforces access control by evaluating policies against `HttpContext.User`. It depends on `UseAuthentication` running first and `UseRouting` having already matched an endpoint. Use `FallbackPolicy` carefully to avoid accidentally locking down public endpoints.

---

## UseSession

**`UseSession`** enables server-side session state, backed by a session store and identified by a session cookie sent to the client. Session data is stored server-side (in-memory by default, or in a distributed cache like Redis or SQL Server).

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddDistributedMemoryCache(); // Required: provides IDistributedCache
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(20);
    options.Cookie.Name = ".OrderPortal.Session";
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true; // Required for GDPR consent
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.SameSite = SameSiteMode.Strict;
});

// Program.cs -- middleware
app.UseSession();
```

### Using Session in Code

```csharp
// Setting session values
public IActionResult AddToCart(int productId)
{
    var cart = HttpContext.Session.GetString("ShoppingCart");
    var cartItems = cart != null 
        ? JsonSerializer.Deserialize<List<int>>(cart) 
        : new List<int>();

    cartItems.Add(productId);

    HttpContext.Session.SetString("ShoppingCart", 
        JsonSerializer.Serialize(cartItems));

    return RedirectToAction("ViewCart");
}

// Getting session values
public IActionResult ViewCart()
{
    var cart = HttpContext.Session.GetString("ShoppingCart");
    var cartItems = cart != null 
        ? JsonSerializer.Deserialize<List<int>>(cart) 
        : new List<int>();

    return View(cartItems);
}
```

### Distributed Session (Redis)

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
    options.InstanceName = "OrderPortal_";
});

builder.Services.AddSession(); // Now backed by Redis
```

### Configuration Options

| Option | Default | Description |
|---|---|---|
| `IdleTimeout` | 20 minutes | Session expires after this period of inactivity |
| `IOTimeout` | 1 minute | Maximum time for loading/committing session |
| `Cookie.Name` | `.AspNetCore.Session` | Name of the session ID cookie |
| `Cookie.HttpOnly` | `true` | Prevents JavaScript access to the cookie |
| `Cookie.IsEssential` | `false` | If `true`, session works even without GDPR consent |
| `Cookie.SecurePolicy` | `None` | Set to `Always` for HTTPS-only |

### When You Need It

When you need to store per-user state on the server between requests (shopping carts, wizard state, user preferences during a session).

### Gotchas

- `AddDistributedMemoryCache()` (or another `IDistributedCache`) **must** be registered before `AddSession()` -- otherwise you get a runtime exception
- The default in-memory cache is **not distributed** -- it does not work across multiple server instances. Use Redis or SQL Server for load-balanced deployments
- Session data is **not loaded automatically**. If you access `HttpContext.Session` without await `LoadAsync()`, it blocks synchronously. In middleware, always call `await HttpContext.Session.LoadAsync()` first
- Session cookies have GDPR implications. Set `IsEssential = true` only if the session is genuinely essential for the application to function

> [!summary] Section Summary
> `UseSession` provides server-side session storage identified by a cookie. Configure the backing store (`IDistributedCache`), cookie settings, and timeout. Use Redis or SQL Server for multi-server deployments. Be mindful of GDPR requirements for session cookies.

---

## UseResponseCaching

**`UseResponseCaching`** caches HTTP responses on the server based on standard HTTP cache headers (`Cache-Control`, `Vary`, etc.). When a cached response is available, the middleware serves it directly without invoking downstream middleware.

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddResponseCaching();

// Program.cs -- middleware
app.UseResponseCaching();
```

### Using the `[ResponseCache]` Attribute

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // Cache for 60 seconds, vary by Accept-Encoding header
    [HttpGet]
    [ResponseCache(Duration = 60, VaryByHeader = "Accept-Encoding")]
    public IActionResult GetProducts()
    {
        var products = _productService.GetAll();
        return Ok(products);
    }

    // Cache for 120 seconds, vary by category query parameter
    [HttpGet("by-category")]
    [ResponseCache(Duration = 120, VaryByQueryKeys = new[] { "category" })]
    public IActionResult GetByCategory([FromQuery] string category)
    {
        var products = _productService.GetByCategory(category);
        return Ok(products);
    }

    // No caching
    [HttpGet("{id}/inventory")]
    [ResponseCache(NoStore = true, Location = ResponseCacheLocation.None)]
    public IActionResult GetInventoryLevel(int id)
    {
        return Ok(_inventoryService.GetLevel(id));
    }
}
```

### Cache Profile Configuration

```csharp
builder.Services.AddControllersWithViews(options =>
{
    options.CacheProfiles.Add("Default60", new CacheProfile
    {
        Duration = 60,
        Location = ResponseCacheLocation.Any,
        VaryByHeader = "Accept-Encoding"
    });

    options.CacheProfiles.Add("NoCache", new CacheProfile
    {
        NoStore = true,
        Location = ResponseCacheLocation.None
    });
});

// Usage:
[ResponseCache(CacheProfileName = "Default60")]
public IActionResult GetProducts() { }
```

### Key Options

| Option | Description |
|---|---|
| `Duration` | Cache duration in seconds |
| `Location` | `Any`, `Client`, or `None` |
| `VaryByHeader` | Cache separate entries based on header value |
| `VaryByQueryKeys` | Cache separate entries based on query string keys |
| `NoStore` | Prevents caching entirely |

### When You Need It

For endpoints that return data which does not change frequently (product catalogs, reference data, public pages).

### Gotchas

- **Does not cache** when the `Authorization` header is present in the request -- this is by design to prevent serving authenticated content to unauthorized users
- **Does not cache** POST, PUT, DELETE, or PATCH requests -- only GET and HEAD
- The response cache middleware respects `Cache-Control: no-cache` and `no-store` from the client
- This is **server-side** caching within the ASP.NET Core process. It is not a CDN or reverse proxy cache. For high-traffic scenarios, consider a dedicated caching layer
- `VaryByQueryKeys` requires the response caching middleware to be in the pipeline -- it does not work with the `[ResponseCache]` attribute alone when relying on client-side caching

> [!summary] Section Summary
> `UseResponseCaching` caches GET/HEAD responses on the server based on HTTP cache headers. Use `[ResponseCache]` attributes or cache profiles for convenience. It does not cache authenticated responses or non-GET methods. For high-scale applications, supplement with a CDN or distributed cache.

---

## UseResponseCompression

**`UseResponseCompression`** compresses HTTP response bodies using algorithms like **gzip** and **Brotli**, reducing the amount of data transferred over the network.

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true; // see security warning below
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
    options.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(
        new[] { "application/json", "text/csv" });
});

builder.Services.Configure<BrotliCompressionProvider>(options =>
{
    options.Level = CompressionLevel.Fastest;
});

builder.Services.Configure<GzipCompressionProvider>(options =>
{
    options.Level = CompressionLevel.SmallestSize;
});

// Program.cs -- middleware
app.UseResponseCompression();
// Place BEFORE UseStaticFiles so static responses are also compressed
app.UseStaticFiles();
```

### Compression Levels

| Level | Behavior | Use Case |
|---|---|---|
| `CompressionLevel.Fastest` | Minimal compression, fast | Real-time APIs |
| `CompressionLevel.Optimal` | Balanced | General use |
| `CompressionLevel.SmallestSize` | Maximum compression, slow | Bandwidth-constrained environments |
| `CompressionLevel.NoCompression` | No compression | Debugging |

### When You Need It

When response body size is a concern and you want to reduce bandwidth usage. Particularly useful for JSON API responses and text-heavy content.

### Gotchas

- **BREACH attack risk**: Enabling compression over HTTPS (`EnableForHttps = true`) can expose your application to the BREACH attack, which exploits compression to extract secrets from encrypted responses. Do not enable this if your responses contain sensitive tokens alongside user-controlled content
- Place `UseResponseCompression` **before** middleware that generates responses (like `UseStaticFiles`), because it needs to intercept the response stream before content is written
- Compression does **not** apply to responses that are already compressed (e.g., images like JPEG and PNG)
- Most reverse proxies (Nginx, IIS, Apache) handle compression at the proxy level. Enabling it in ASP.NET Core as well can cause double-compression overhead. Decide where compression should live in your architecture

> [!warning]
> If your reverse proxy already handles compression, **do not enable it again** in ASP.NET Core. Double-compressing wastes CPU and provides negligible size reduction.

> [!summary] Section Summary
> `UseResponseCompression` reduces response size using gzip or Brotli. Configure compression providers and levels based on your performance needs. Be aware of the BREACH attack risk over HTTPS and avoid double-compression when behind a reverse proxy.

---

## UseRateLimiter

**`UseRateLimiter`** (introduced in **.NET 7**) provides built-in rate limiting to protect your application from excessive requests. It supports four algorithms: **fixed window**, **sliding window**, **token bucket**, and **concurrency limiter**.

### Service Registration

```csharp
using System.Threading.RateLimiting;

builder.Services.AddRateLimiter(options =>
{
    // Global rejection behavior
    options.RejectionStatusCode = StatusCodes.Status429TooManyRequests;

    // When a request is rejected, add Retry-After header
    options.OnRejected = async (context, cancellationToken) =>
    {
        context.HttpContext.Response.Headers.RetryAfter = "60";
        await context.HttpContext.Response.WriteAsJsonAsync(new ProblemDetails
        {
            Status = 429,
            Title = "Too Many Requests",
            Detail = "Rate limit exceeded. Please retry after 60 seconds."
        }, cancellationToken);
    };
});
```

### Fixed Window Limiter

Allows a fixed number of requests within a time window. The window resets completely when the time expires.

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("FixedPolicy", opt =>
    {
        opt.PermitLimit = 100;
        opt.Window = TimeSpan.FromMinutes(1);
        opt.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;
        opt.QueueLimit = 10; // queue 10 extra requests instead of rejecting
    });
});
```

> [!info]
> **Fixed window** is the simplest algorithm. The downside is the "burst at boundary" problem: if a client sends 100 requests at the end of window 1 and 100 more at the start of window 2, they effectively send 200 requests in a very short period. Use **sliding window** to mitigate this.

### Sliding Window Limiter

Divides the window into segments and smoothly slides the limit, preventing boundary burst problems.

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddSlidingWindowLimiter("SlidingPolicy", opt =>
    {
        opt.PermitLimit = 100;
        opt.Window = TimeSpan.FromMinutes(1);
        opt.SegmentsPerWindow = 6; // 10-second segments
        opt.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;
        opt.QueueLimit = 5;
    });
});
```

### Token Bucket Limiter

Tokens are added to a bucket at a fixed rate. Each request consumes one token. When the bucket is empty, requests are rejected. This naturally allows short bursts while enforcing an average rate.

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddTokenBucketLimiter("TokenBucketPolicy", opt =>
    {
        opt.TokenLimit = 50;           // max tokens in bucket
        opt.ReplenishmentPeriod = TimeSpan.FromSeconds(10);
        opt.TokensPerPeriod = 10;      // add 10 tokens every 10 seconds
        opt.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;
        opt.QueueLimit = 5;
        opt.AutoReplenishment = true;
    });
});
```

### Concurrency Limiter

Limits the number of **concurrent** requests rather than requests over time. Useful for protecting resource-intensive endpoints.

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddConcurrencyLimiter("ConcurrencyPolicy", opt =>
    {
        opt.PermitLimit = 20;          // max 20 concurrent requests
        opt.QueueProcessingOrder = QueueProcessingOrder.OldestFirst;
        opt.QueueLimit = 10;
    });
});
```

### Applying Rate Limiters to Endpoints

```csharp
// Program.cs -- middleware
app.UseRateLimiter();

// Apply to minimal API endpoints
app.MapGet("/api/orders", () => Results.Ok())
    .RequireRateLimiting("SlidingPolicy");

// Apply to controllers via attribute
[EnableRateLimiting("TokenBucketPolicy")]
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [DisableRateLimiting] // Exempt this endpoint
    [HttpGet("health")]
    public IActionResult HealthCheck() => Ok();

    [HttpPost]
    public IActionResult CreateOrder(OrderRequest request)
    {
        // Rate-limited by TokenBucketPolicy
        return Ok();
    }
}
```

### Rate Limiting by Client IP

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddPolicy("PerClientPolicy", context =>
    {
        var clientIp = context.Connection.RemoteIpAddress?.ToString() ?? "unknown";

        return RateLimitPartition.GetFixedWindowLimiter(clientIp, _ =>
            new FixedWindowRateLimiterOptions
            {
                PermitLimit = 60,
                Window = TimeSpan.FromMinutes(1)
            });
    });
});
```

### Algorithm Comparison

| Algorithm | Best For | Burst Handling | Complexity |
|---|---|---|---|
| Fixed Window | Simple rate limits | Poor (boundary burst) | Low |
| Sliding Window | Smooth rate enforcement | Good | Medium |
| Token Bucket | APIs with occasional burst tolerance | Excellent | Medium |
| Concurrency | Resource-intensive endpoints | N/A (concurrent, not time-based) | Low |

### When You Need It

APIs exposed to the public internet, multi-tenant applications, or any endpoint where you need to prevent abuse or protect backend resources.

### Gotchas

- You **must** call `builder.Services.AddRateLimiter()` to register the service -- `app.UseRateLimiter()` alone throws a runtime exception
- Rate limiters are **per-server instance** by default. In a load-balanced environment, each server tracks its own counts. For distributed rate limiting, use an external store (Redis) or an API gateway
- The `QueueLimit` determines how many excess requests wait in a queue instead of being immediately rejected. Setting it too high can cause memory pressure under heavy load
- Place `UseRateLimiter` **after** `UseRouting` so that per-endpoint policies can be resolved based on the matched endpoint

> [!summary] Section Summary
> `UseRateLimiter` (.NET 7+) provides four algorithms: fixed window, sliding window, token bucket, and concurrency limiter. Apply policies globally or per-endpoint. Rate limiting is per-server-instance by default -- use external stores for distributed scenarios. Always register the service with `AddRateLimiter` before using the middleware.

---

## Comprehensive Summary

> [!tip] Complete Summary
> ASP.NET Core's built-in middleware components form a layered pipeline that handles cross-cutting concerns before requests reach your application logic.
>
> **Error handling** -- `UseExceptionHandler` (production) and `UseDeveloperExceptionPage` (development) ensure exceptions are caught and handled appropriately. Never expose developer error pages in production.
>
> **Security** -- `UseHsts` and `UseHttpsRedirection` enforce HTTPS. `UseAuthentication` identifies users, `UseAuthorization` enforces access control, and `UseCors` controls cross-origin access. The order between these is critical.
>
> **Performance** -- `UseStaticFiles` short-circuits for static content, `UseResponseCaching` caches responses based on HTTP headers, `UseResponseCompression` reduces bandwidth with gzip/Brotli, and `UseRateLimiter` protects against excessive traffic.
>
> **Routing** -- `UseRouting` is the central hub that matches URLs to endpoints, enabling all metadata-aware middleware (auth, CORS, rate limiting) to function correctly.
>
> **Session** -- `UseSession` provides server-side state storage backed by distributed cache providers.
>
> The single most important principle across all middleware is **order matters**. The recommended pipeline order (exception handling, HTTPS, static files, routing, CORS, authentication, authorization, session, caching, rate limiting, endpoint execution) ensures each component has the context it needs from earlier components. Placing middleware out of order is the most common cause of subtle bugs in ASP.NET Core applications.

---

## Related Topics

- [[Middleware Overview]] -- foundational concepts of the request pipeline and how middleware works
- [[Request Pipeline]] -- the full request/response flow through the middleware chain
- [[Custom Middleware]] -- writing your own middleware components
- [[Authentication and Authorization]] -- deeper dive into identity and access control
- [[Routing]] -- endpoint routing internals and route templates
- [[Error Handling and Logging]] -- structured error handling strategies
- [[Dependency Injection]] -- how middleware integrates with the DI container

---

## Further Reading

- [[ASP.NET Core Fundamentals]] -- overview of the framework architecture
- [[Configuration]] -- managing app settings that middleware depends on
- [[Web APIs]] -- building APIs that leverage these middleware components
- [[Controllers and Actions]] -- MVC controller patterns that interact with routing and authorization
