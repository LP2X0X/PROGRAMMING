---
tags:
 - csharp
 - asp-net-core
 - filters
 - controllers
 - cross-cutting
---

# Filters

Filters are code that runs **before and/or after** specific stages of the MVC request pipeline. They act as hooks into the action execution process, letting you implement **cross-cutting concerns** (logging, authorization, caching, exception handling) without duplicating code in every action method.

```ad-note
Filters only apply to requests that reach the **MVC/Razor Pages pipeline**. Requests for static files or non-MVC endpoints never hit filters. This is a key distinction from [[17.02 - Middleware]], which runs for ALL requests.
```

---

## Filter Pipeline -- Execution Order

Filters execute in a strict, well-defined order. Each filter type has a "before" hook and an "after" hook, forming a layered pipeline around the action method:

```
Authorization Filter
  Resource Filter (before)
    Model Binding
      Action Filter (before)
        --- ACTION EXECUTES ---
      Action Filter (after)
    Exception Filter (if exception thrown)
    Result Filter (before)
      --- RESULT EXECUTES ---
    Result Filter (after)
  Resource Filter (after)
```

The five filter types, in execution order:

| Order | Filter Type | Interface | Purpose |
|-------|------------|-----------|---------|
| 1 | Authorization | `IAuthorizationFilter` | Gate access, run first |
| 2 | Resource | `IResourceFilter` | Caching, short-circuit before model binding |
| 3 | Action | `IActionFilter` | Inspect/modify arguments, wrap action execution |
| 4 | Exception | `IExceptionFilter` | Handle unhandled exceptions from the action |
| 5 | Result | `IResultFilter` | Wrap action result execution |

```ad-attention
The "after" hooks run in **reverse order** (inside-out). If the action throws, the exception filter runs instead of the normal action-after and result filters. Resource filter's "after" hook always runs last, even after result execution.
```

---

## Authorization Filters

Authorization filters run **first** in the pipeline, before anything else. They decide whether the current user is allowed to proceed.

- **Interface**: `IAuthorizationFilter` / `IAsyncAuthorizationFilter`
- Can **short-circuit** the entire pipeline by setting `context.Result`
- Run before model binding, before resource filters

```csharp
public class RequireClaimFilter : IAuthorizationFilter
{
    private readonly string _claimType;

    public RequireClaimFilter(string claimType)
    {
        _claimType = claimType;
    }

    public void OnAuthorization(AuthorizationFilterContext context)
    {
        bool hasClaim = context.HttpContext.User.HasClaim(c => c.Type == _claimType);

        if (!hasClaim)
        {
            context.Result = new ForbidResult(); // Short-circuits the pipeline
        }
    }
}
```

```ad-warning
You should **rarely** implement custom authorization filters. The built-in policy-based authorization system is far more flexible, testable, and maintainable. Use `[Authorize(Policy = "...")]` instead.

See [[17.09 - Authentication and Authorization]] for the policy-based approach.
```

---

## Resource Filters

Resource filters run **after authorization** but **before model binding**. They wrap the entire rest of the pipeline, so their "after" hook runs last.

- **Interface**: `IResourceFilter` / `IAsyncResourceFilter`
- Two hooks: `OnResourceExecuting` (before model binding) and `OnResourceExecuted` (after everything)
- Use cases: response caching, short-circuiting expensive operations, resource cleanup

### Caching Resource Filter Example

```csharp
public class CacheResourceFilter : IResourceFilter
{
    private readonly IMemoryCache _cache;

    public CacheResourceFilter(IMemoryCache cache)
    {
        _cache = cache;
    }

    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        string cacheKey = context.HttpContext.Request.Path.ToString();

        if (_cache.TryGetValue(cacheKey, out IActionResult? cachedResult))
        {
            // Short-circuit: return cached result, skip the rest of the pipeline
            context.Result = cachedResult!;
        }
    }

    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        if (context.Result is not null)
        {
            string cacheKey = context.HttpContext.Request.Path.ToString();

            var cacheOptions = new MemoryCacheEntryOptions()
                .SetAbsoluteExpiration(TimeSpan.FromMinutes(5));

            _cache.Set(cacheKey, context.Result, cacheOptions);
        }
    }
}
```

```ad-tip
Resource filters are ideal for caching because they wrap the entire pipeline. Setting `context.Result` in `OnResourceExecuting` short-circuits model binding, action execution, and everything else -- returning the cached response directly.
```

---

## Action Filters

Action filters are the **most commonly used** filter type. They run after model binding, wrapping the action method execution itself.

- **Interface**: `IActionFilter` / `IAsyncActionFilter`
- `OnActionExecuting` -- runs after model binding, before the action
- `OnActionExecuted` -- runs after the action method completes
- Can inspect/modify action arguments, modify the result, or short-circuit execution

### Synchronous Implementation

```csharp
public class LogActionFilter : IActionFilter
{
    private readonly ILogger<LogActionFilter> _logger;

    public LogActionFilter(ILogger<LogActionFilter> logger)
    {
        _logger = logger;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        string actionName = context.ActionDescriptor.DisplayName ?? "Unknown";
        string arguments = JsonSerializer.Serialize(context.ActionArguments);

        _logger.LogInformation(
            "Executing action {Action} with arguments: {Arguments}",
            actionName,
            arguments);
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        string actionName = context.ActionDescriptor.DisplayName ?? "Unknown";

        if (context.Exception is not null)
        {
            _logger.LogError(context.Exception,
                "Action {Action} threw an exception", actionName);
        }
        else
        {
            _logger.LogInformation("Action {Action} executed successfully", actionName);
        }
    }
}
```

### Asynchronous Implementation

```csharp
public class AsyncLogActionFilter : IAsyncActionFilter
{
    private readonly ILogger<AsyncLogActionFilter> _logger;

    public AsyncLogActionFilter(ILogger<AsyncLogActionFilter> logger)
    {
        _logger = logger;
    }

    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        string actionName = context.ActionDescriptor.DisplayName ?? "Unknown";

        _logger.LogInformation("Before executing action {Action}", actionName);

        // Call next() to execute the action (and any remaining filters)
        ActionExecutedContext resultContext = await next();

        if (resultContext.Exception is not null)
        {
            _logger.LogError(resultContext.Exception,
                "Action {Action} threw an exception", actionName);
        }
        else
        {
            _logger.LogInformation("After executing action {Action}", actionName);
        }
    }
}
```

```ad-note
The async version uses a single method with an `ActionExecutionDelegate next` parameter. You call `await next()` to execute the action. Everything before `next()` is the "before" logic, everything after is the "after" logic. If you never call `next()`, the action is short-circuited.
```

### Short-Circuiting from an Action Filter

```csharp
public void OnActionExecuting(ActionExecutingContext context)
{
    if (!context.ModelState.IsValid)
    {
        // Short-circuit: skip the action entirely and return a 400
        context.Result = new BadRequestObjectResult(context.ModelState);
    }
}
```

---

## Exception Filters

Exception filters catch **unhandled exceptions** thrown by the action method, action filters, or model binding. They do NOT catch exceptions from authorization filters, resource filters, or result filters.

- **Interface**: `IExceptionFilter` / `IAsyncExceptionFilter`
- Set `context.ExceptionHandled = true` to suppress the exception
- Set `context.Result` to provide a custom error response

```csharp
public class ApiExceptionFilter : IExceptionFilter
{
    private readonly ILogger<ApiExceptionFilter> _logger;
    private readonly IHostEnvironment _environment;

    public ApiExceptionFilter(
        ILogger<ApiExceptionFilter> logger,
        IHostEnvironment environment)
    {
        _logger = logger;
        _environment = environment;
    }

    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception,
            "Unhandled exception in action {Action}",
            context.ActionDescriptor.DisplayName);

        var problemDetails = new ProblemDetails
        {
            Title = "An error occurred",
            Status = StatusCodes.Status500InternalServerError
        };

        // Map known exception types to appropriate HTTP status codes
        switch (context.Exception)
        {
            case KeyNotFoundException:
                problemDetails.Title = "Resource not found";
                problemDetails.Status = StatusCodes.Status404NotFound;
                break;

            case UnauthorizedAccessException:
                problemDetails.Title = "Access denied";
                problemDetails.Status = StatusCodes.Status403Forbidden;
                break;

            case ArgumentException:
                problemDetails.Title = "Invalid request";
                problemDetails.Status = StatusCodes.Status400BadRequest;
                break;
        }

        // Include exception details only in development
        if (_environment.IsDevelopment())
        {
            problemDetails.Detail = context.Exception.ToString();
        }

        context.Result = new ObjectResult(problemDetails)
        {
            StatusCode = problemDetails.Status
        };

        context.ExceptionHandled = true;
    }
}
```

```ad-warning
Exception filters are **not** a replacement for global exception handling middleware. Middleware-based exception handling (like `UseExceptionHandler`) catches exceptions from the entire pipeline, including middleware. Exception filters only catch exceptions from MVC-specific stages. Use **both** for defense-in-depth.
```

---

## Result Filters

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

---

## Implementing Filters

There are three main approaches to applying filters, each with different trade-offs around dependency injection support.

### As Attributes (Simple)

Inherit from a base attribute class that implements the filter interface. These classes are both attributes and filters, so they can be applied directly with `[SquareBracket]` syntax.

```csharp
public class SimpleLogFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        // No access to DI services here -- cannot inject via constructor
        Debug.WriteLine($"Executing: {context.ActionDescriptor.DisplayName}");
    }

    public override void OnActionExecuted(ActionExecutedContext context)
    {
        Debug.WriteLine($"Executed: {context.ActionDescriptor.DisplayName}");
    }
}

// Usage
[SimpleLogFilter]
public class ProductsController : Controller
{
    [SimpleLogFilter] // Can also apply at action level
    public IActionResult GetAll() => Ok();
}
```

Available base attribute classes:
- `ActionFilterAttribute` -- implements both `IActionFilter` and `IResultFilter`
- `ExceptionFilterAttribute` -- implements `IExceptionFilter`
- `ResultFilterAttribute` -- implements `IResultFilter`

```ad-warning
The attribute approach **cannot use constructor dependency injection**. Attribute constructors only accept compile-time constants. If your filter needs injected services (loggers, database contexts, configuration), use `[TypeFilter]` or `[ServiceFilter]` instead.
```

### TypeFilter -- DI-Resolved via Attribute

`[TypeFilter]` tells the framework to create the filter using the DI container, resolving constructor dependencies automatically.

```csharp
public class LogActionFilter : IActionFilter
{
    private readonly ILogger<LogActionFilter> _logger;
    private readonly string _source;

    // ILogger injected from DI, source passed via Arguments
    public LogActionFilter(ILogger<LogActionFilter> logger, string source)
    {
        _logger = logger;
        _source = source;
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        _logger.LogInformation("[{Source}] Executing: {Action}",
            _source, context.ActionDescriptor.DisplayName);
    }

    public void OnActionExecuted(ActionExecutedContext context) { }
}

// Usage -- DI resolves ILogger, "Catalog" passed as extra argument
[TypeFilter(typeof(LogActionFilter), Arguments = new object[] { "Catalog" })]
public class CatalogController : Controller
{
    // ...
}
```

```ad-note
`TypeFilter` does **not** require the filter to be registered in the DI container. It creates the instance on the fly, resolving constructor parameters from the container where possible and from `Arguments` for the rest.
```

### ServiceFilter -- DI-Resolved from Services

`[ServiceFilter]` resolves the filter directly from the DI container. The filter **must** be explicitly registered as a service.

```csharp
// Register in Program.cs
builder.Services.AddScoped<PerformanceTimingFilter>();

// The filter class (same as any DI-resolved filter)
public class PerformanceTimingFilter : IActionFilter
{
    private readonly ILogger<PerformanceTimingFilter> _logger;

    public PerformanceTimingFilter(ILogger<PerformanceTimingFilter> logger)
    {
        _logger = logger;
    }

    // ...filter methods...
}

// Usage
[ServiceFilter(typeof(PerformanceTimingFilter))]
public class OrdersController : Controller
{
    // ...
}
```

```ad-tip
`ServiceFilter` is more explicit than `TypeFilter` -- the filter must be registered, so you get a clear error at startup if the registration is missing. It also gives you control over the service lifetime (transient, scoped, singleton).
```

### Comparison Table

| Approach | DI Support | Registration Required | Extra Arguments | Best For |
|----------|-----------|----------------------|-----------------|----------|
| Attribute | No | No | No | Simple filters with no dependencies |
| TypeFilter | Yes | No | Yes (`Arguments`) | Filters needing DI + extra config |
| ServiceFilter | Yes | Yes (explicit) | No | Filters already registered in DI |

---

## Global Filters

Global filters apply to **every controller and every action** without needing attributes. Register them in `Program.cs`:

```csharp
builder.Services.AddControllers(options =>
{
    // Add filter by type (resolved from DI if registered, otherwise activated)
    options.Filters.Add<LogActionFilter>();
    options.Filters.Add<ApiExceptionFilter>();

    // Add filter instance directly (must be parameterless or pre-constructed)
    options.Filters.Add(new ResponseHeaderFilter());

    // Add with explicit order
    options.Filters.Add<PerformanceTimingFilter>(order: 1);
});
```

```ad-note
Global filters are useful for concerns that truly apply everywhere: exception handling, performance logging, security headers, etc. For concerns that apply to a subset of actions, prefer attribute-based application.
```

---

## Filter Ordering

### Order by Filter Type

Filters always execute in type order regardless of registration:

1. Authorization
2. Resource
3. Action
4. Exception
5. Result

### Order Within the Same Type

Within the same filter type, the default execution order is:

1. **Global** filters (registered in `AddControllers`)
2. **Controller-level** filters (attribute on the controller class)
3. **Action-level** filters (attribute on the action method)

The "after" hooks run in **reverse** (action -> controller -> global).

### Custom Ordering with IOrderedFilter

Override the default scope-based ordering by implementing `IOrderedFilter`:

```csharp
public class AuditLogFilter : ActionFilterAttribute, IOrderedFilter
{
    // Lower values execute first. Default is 0.
    public new int Order { get; set; } = -1; // Run before other action filters
}

public class ValidationFilter : ActionFilterAttribute, IOrderedFilter
{
    public new int Order { get; set; } = 0;
}

public class CachingFilter : ActionFilterAttribute, IOrderedFilter
{
    public new int Order { get; set; } = 1; // Run after audit and validation
}
```

```csharp
// Or set order when registering globally
builder.Services.AddControllers(options =>
{
    options.Filters.Add<AuditLogFilter>(order: -1);
    options.Filters.Add<ValidationFilter>(order: 0);
    options.Filters.Add<CachingFilter>(order: 1);
});
```

```ad-note
When two filters have the **same** `Order` value, the scope-based rule applies (global before controller before action). Lower `Order` values always run their "before" hook first, and their "after" hook last.
```

---

## Commonly Used Built-in Attributes

ASP.NET Core ships with many filter attributes out of the box. These cover the most common cross-cutting concerns.

### Authorization

```csharp
// Require any authenticated user
[Authorize]
public class AccountController : Controller { }

// Require specific role
[Authorize(Roles = "Admin,Manager")]
public IActionResult DeleteUser(int id) => Ok();

// Require a named policy
[Authorize(Policy = "CanEditProducts")]
public IActionResult Edit(int id) => Ok();

// Override authorization on a specific action
[Authorize]
public class AdminController : Controller
{
    [AllowAnonymous] // Anyone can access this action
    public IActionResult PublicStatus() => Ok("Running");
}
```

See [[17.09 - Authentication and Authorization]] for defining policies.

### HTTPS and Transport Security

```csharp
// Redirect HTTP requests to HTTPS
[RequireHttps]
public class SecureController : Controller { }
```

### Response Caching

```csharp
// Cache the response for 60 seconds
[ResponseCache(Duration = 60)]
public IActionResult GetProducts() => Ok(products);

// Cache with varying by query string
[ResponseCache(Duration = 120, VaryByQueryKeys = new[] { "category", "page" })]
public IActionResult Search(string category, int page) => Ok();

// No caching
[ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
public IActionResult GetSensitiveData() => Ok();
```

### Anti-Forgery (CSRF Protection)

```csharp
// Require anti-forgery token for this POST action
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult SubmitForm(FormModel model) => Ok();

// Skip anti-forgery validation (e.g., for API endpoints using JWT)
[HttpPost]
[IgnoreAntiforgeryToken]
public IActionResult ApiEndpoint([FromBody] RequestDto dto) => Ok();
```

### Content Negotiation

```csharp
// Only accept JSON requests
[Consumes("application/json")]
[HttpPost]
public IActionResult Create([FromBody] ProductDto product) => Ok();

// Declare that this action always returns JSON
[Produces("application/json")]
public IActionResult GetAll() => Ok(products);

// Enable URL-based format selection (/api/products.json, /api/products.xml)
[FormatFilter]
[Route("api/products.{format?}")]
public IActionResult Get() => Ok(products);
```

---

## Filters vs Middleware

Understanding when to use filters versus [[17.02 - Middleware]] is important. They serve overlapping but distinct roles.

| Aspect | Middleware | Filters |
|--------|-----------|---------|
| Scope | Every HTTP request | Only MVC/Razor Pages requests |
| Context | `HttpContext` only | `ActionContext`, `ModelState`, controller, arguments |
| Ordering | Pipeline order in `Program.cs` | Type-based + scope-based + `Order` |
| Short-circuit | Return without calling `next()` | Set `context.Result` |
| DI | Constructor injection | Constructor injection (via TypeFilter/ServiceFilter) |

### When to Use Middleware

- Authentication / authorization schemes
- CORS
- Static file serving
- Request logging (all requests, including non-MVC)
- Global exception handling (`UseExceptionHandler`)
- Request/response compression
- Rate limiting

### When to Use Filters

- Action-specific authorization policies
- Model validation logic
- Action-specific logging (with access to action arguments)
- Response modification (headers, wrapping)
- MVC-specific exception handling
- Caching for specific actions

```ad-tip
A good rule of thumb: if you need access to MVC-specific information (action name, model state, action arguments, controller instance), use a filter. If the concern applies to all HTTP traffic regardless of whether MVC handles it, use middleware.
```

---

## Real-World Examples

### 1. Audit Logging Filter

Logs who called what action, with what parameters, at what time. Includes the authenticated user identity from claims.

```csharp
public class AuditLogFilter : IAsyncActionFilter
{
    private readonly ILogger<AuditLogFilter> _logger;

    public AuditLogFilter(ILogger<AuditLogFilter> logger)
    {
        _logger = logger;
    }

    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        var httpContext = context.HttpContext;
        var user = httpContext.User;

        string userId = user.FindFirst(ClaimTypes.NameIdentifier)?.Value ?? "anonymous";
        string userName = user.Identity?.Name ?? "unknown";
        string actionName = context.ActionDescriptor.DisplayName ?? "Unknown";
        string httpMethod = httpContext.Request.Method;
        string path = httpContext.Request.Path;
        string ipAddress = httpContext.Connection.RemoteIpAddress?.ToString() ?? "unknown";

        // Serialize action arguments, excluding sensitive types
        var safeArguments = context.ActionArguments
            .Where(kvp => kvp.Value is not IFormFile and not IFormFileCollection)
            .ToDictionary(kvp => kvp.Key, kvp => kvp.Value);
        string arguments = JsonSerializer.Serialize(safeArguments);

        _logger.LogInformation(
            "AUDIT: User={UserId} ({UserName}) | {Method} {Path} | Action={Action} | Args={Arguments} | IP={IP} | Time={Time}",
            userId,
            userName,
            httpMethod,
            path,
            actionName,
            arguments,
            ipAddress,
            DateTime.UtcNow);

        ActionExecutedContext resultContext = await next();

        if (resultContext.Exception is not null)
        {
            _logger.LogWarning(
                "AUDIT: Action={Action} | User={UserId} | FAILED with {ExceptionType}: {Message}",
                actionName,
                userId,
                resultContext.Exception.GetType().Name,
                resultContext.Exception.Message);
        }
        else
        {
            int? statusCode = (resultContext.Result as ObjectResult)?.StatusCode
                ?? (resultContext.Result as StatusCodeResult)?.StatusCode;

            _logger.LogInformation(
                "AUDIT: Action={Action} | User={UserId} | Completed with status {StatusCode}",
                actionName,
                userId,
                statusCode);
        }
    }
}
```

Register globally:

```csharp
builder.Services.AddScoped<AuditLogFilter>();

builder.Services.AddControllers(options =>
{
    options.Filters.Add<AuditLogFilter>();
});
```

### 2. Performance Timing Filter

Measures how long the action method takes to execute. Logs a warning if it exceeds a configurable threshold.

```csharp
public class PerformanceTimingFilter : IAsyncActionFilter
{
    private readonly ILogger<PerformanceTimingFilter> _logger;
    private readonly TimeSpan _threshold;

    public PerformanceTimingFilter(
        ILogger<PerformanceTimingFilter> logger,
        IConfiguration configuration)
    {
        _logger = logger;

        int thresholdMs = configuration.GetValue<int>(
            "PerformanceMonitoring:SlowActionThresholdMs", defaultValue: 500);
        _threshold = TimeSpan.FromMilliseconds(thresholdMs);
    }

    public async Task OnActionExecutionAsync(
        ActionExecutingContext context,
        ActionExecutionDelegate next)
    {
        string actionName = context.ActionDescriptor.DisplayName ?? "Unknown";
        var stopwatch = Stopwatch.StartNew();

        ActionExecutedContext resultContext = await next();

        stopwatch.Stop();
        TimeSpan elapsed = stopwatch.Elapsed;

        if (elapsed > _threshold)
        {
            _logger.LogWarning(
                "SLOW ACTION: {Action} took {ElapsedMs}ms (threshold: {ThresholdMs}ms)",
                actionName,
                elapsed.TotalMilliseconds,
                _threshold.TotalMilliseconds);
        }
        else
        {
            _logger.LogDebug(
                "Action {Action} completed in {ElapsedMs}ms",
                actionName,
                elapsed.TotalMilliseconds);
        }

        // Optionally add timing as a response header
        if (!context.HttpContext.Response.HasStarted)
        {
            context.HttpContext.Response.Headers["X-Action-Duration-Ms"] =
                elapsed.TotalMilliseconds.ToString("F2");
        }
    }
}
```

Configuration in `appsettings.json`:

```json
{
  "PerformanceMonitoring": {
    "SlowActionThresholdMs": 500
  }
}
```

Register:

```csharp
builder.Services.AddScoped<PerformanceTimingFilter>();
builder.Services.AddControllers(options =>
{
    options.Filters.Add<PerformanceTimingFilter>();
});
```

### 3. API Key Validation Filter

Reads an API key from the request header, validates it against configuration, and returns 401 if missing or invalid.

```csharp
public class ApiKeyAuthFilter : IAuthorizationFilter
{
    private const string ApiKeyHeaderName = "X-Api-Key";

    private readonly IConfiguration _configuration;
    private readonly ILogger<ApiKeyAuthFilter> _logger;

    public ApiKeyAuthFilter(
        IConfiguration configuration,
        ILogger<ApiKeyAuthFilter> logger)
    {
        _configuration = configuration;
        _logger = logger;
    }

    public void OnAuthorization(AuthorizationFilterContext context)
    {
        // Check if the action has [AllowAnonymous] -- skip validation if so
        if (context.ActionDescriptor.EndpointMetadata
            .OfType<AllowAnonymousAttribute>().Any())
        {
            return;
        }

        // Try to extract the API key from the header
        if (!context.HttpContext.Request.Headers
            .TryGetValue(ApiKeyHeaderName, out var extractedKey))
        {
            _logger.LogWarning(
                "API key missing from request to {Path}",
                context.HttpContext.Request.Path);

            context.Result = new UnauthorizedObjectResult(new ProblemDetails
            {
                Title = "API key missing",
                Detail = $"Provide a valid API key in the '{ApiKeyHeaderName}' header.",
                Status = StatusCodes.Status401Unauthorized
            });
            return;
        }

        // Validate against configured keys
        string? configuredKey = _configuration.GetValue<string>("ApiSecurity:ApiKey");

        if (configuredKey is null || !string.Equals(
            extractedKey, configuredKey, StringComparison.Ordinal))
        {
            _logger.LogWarning(
                "Invalid API key provided for request to {Path}",
                context.HttpContext.Request.Path);

            context.Result = new UnauthorizedObjectResult(new ProblemDetails
            {
                Title = "Invalid API key",
                Detail = "The provided API key is not valid.",
                Status = StatusCodes.Status401Unauthorized
            });
        }
    }
}
```

Configuration in `appsettings.json`:

```json
{
  "ApiSecurity": {
    "ApiKey": "your-secret-api-key-here"
  }
}
```

Apply to specific controllers using `[TypeFilter]` or register globally:

```csharp
// Option A: Apply to a specific controller
[TypeFilter(typeof(ApiKeyAuthFilter))]
[ApiController]
[Route("api/[controller]")]
public class ExternalDataController : ControllerBase
{
    [HttpGet]
    public IActionResult GetData() => Ok(new { value = "secret data" });

    [AllowAnonymous]
    public IActionResult Health() => Ok("Healthy"); // Skips API key check
}

// Option B: Register globally for all controllers
builder.Services.AddControllers(options =>
{
    options.Filters.Add<ApiKeyAuthFilter>();
});
```

```ad-tip
For production API key management, consider using a dedicated authentication handler rather than an authorization filter. Filters work well for simple scenarios, but authentication handlers integrate properly with the ASP.NET Core authentication/authorization pipeline.
```

---

## See Also

- [[Controllers Overview]] -- how controllers and actions work
- [[Action Results]] -- the IActionResult types that filters wrap around
- [[Model Binding]] -- how request data becomes action parameters (runs between resource and action filters)
- [[Validation]] -- model validation that filters can inspect via `ModelState`
- [[17.02 - Middleware]] -- request pipeline middleware (runs before filters)
- [[17.09 - Authentication and Authorization]] -- policy-based authorization (preferred over custom auth filters)
