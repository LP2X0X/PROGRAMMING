---
tags:
  - csharp
  - asp-net-core
  - filters
  - controllers
  - cross-cutting
---


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
