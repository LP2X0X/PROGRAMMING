---
tags:
  - csharp
  - asp-net-core
  - middleware
  - custom
---

## Real-World Examples

### Request Timing Middleware

Measures how long each request takes and adds the elapsed time as a response header. Useful for performance monitoring.

```csharp
using System.Diagnostics;

public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestTimingMiddleware> _logger;

    public RequestTimingMiddleware(
        RequestDelegate next,
        ILogger<RequestTimingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();

        // Register a callback that fires just before the response headers are sent.
        // This is necessary because once _next(context) returns,
        // the response may have already started streaming.
        context.Response.OnStarting(() =>
        {
            stopwatch.Stop();
            context.Response.Headers["X-Response-Time"] =
                $"{stopwatch.ElapsedMilliseconds}ms";
            return Task.CompletedTask;
        });

        try
        {
            await _next(context);
        }
        finally
        {
            stopwatch.Stop();
            _logger.LogInformation(
                "{Method} {Path} completed in {ElapsedMs}ms with status {StatusCode}",
                context.Request.Method,
                context.Request.Path,
                stopwatch.ElapsedMilliseconds,
                context.Response.StatusCode);
        }
    }
}

public static class RequestTimingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestTiming(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestTimingMiddleware>();
    }
}
```

> [!ad-note]
> `Response.OnStarting()` is used to set the header because by the time `await _next(context)` returns, the response headers may have already been sent to the client. `OnStarting` fires just before the headers are flushed, giving us the last opportunity to modify them.

> [!summary] Section Summary
> Request timing middleware wraps the `_next` call with a `Stopwatch`, uses `Response.OnStarting` to inject a header before the response is flushed, and logs the duration. This is a fundamental observability tool for any production API.

### Correlation ID Middleware

Generates or propagates a unique **correlation ID** for each request. This ID is used to correlate log entries across services in distributed systems.

```csharp
public class CorrelationIdMiddleware
{
    private const string CorrelationIdHeaderName = "X-Correlation-Id";
    private readonly RequestDelegate _next;
    private readonly ILogger<CorrelationIdMiddleware> _logger;

    public CorrelationIdMiddleware(
        RequestDelegate next,
        ILogger<CorrelationIdMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Check if the caller already provided a correlation ID
        // (common in service-to-service calls)
        if (!context.Request.Headers.TryGetValue(
                CorrelationIdHeaderName, out var correlationId)
            || string.IsNullOrWhiteSpace(correlationId))
        {
            correlationId = Guid.NewGuid().ToString("N");
        }

        // Store it in HttpContext.Items so downstream code can access it
        context.Items["CorrelationId"] = correlationId.ToString();

        // Add a log scope so all log entries include the correlation ID
        using (_logger.BeginScope(
            new Dictionary<string, object>
            {
                ["CorrelationId"] = correlationId.ToString()
            }))
        {
            _logger.LogInformation(
                "Request {Method} {Path} with CorrelationId {CorrelationId}",
                context.Request.Method,
                context.Request.Path,
                correlationId);

            // Echo the correlation ID back in the response
            context.Response.OnStarting(() =>
            {
                context.Response.Headers[CorrelationIdHeaderName] =
                    correlationId.ToString();
                return Task.CompletedTask;
            });

            await _next(context);
        }
    }
}

public static class CorrelationIdMiddlewareExtensions
{
    public static IApplicationBuilder UseCorrelationId(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<CorrelationIdMiddleware>();
    }
}
```

Usage:

```csharp
var app = builder.Build();

app.UseCorrelationId(); // Should be early in the pipeline
app.UseRequestTiming();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

Accessing the correlation ID in a controller:

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpGet("{orderId}")]
    public IActionResult GetOrder(int orderId)
    {
        var correlationId = HttpContext.Items["CorrelationId"]?.ToString();
        // Use correlationId in downstream service calls, logging, etc.
        return Ok(new { OrderId = orderId, CorrelationId = correlationId });
    }
}
```

> [!tip]
> Place correlation ID middleware as early as possible in the pipeline so that all subsequent middleware and handlers have access to the ID for logging and tracing.

> [!summary] Section Summary
> Correlation ID middleware checks for an existing `X-Correlation-Id` header, generates one if missing, stores it in `HttpContext.Items`, wraps downstream processing in a log scope, and echoes the ID back in the response. Essential for distributed tracing.

### API Key Authentication Middleware

Validates an **API key** from the `X-Api-Key` request header against a configured value. Rejects unauthorized requests before they reach controllers.

```csharp
public class ApiKeyMiddleware
{
    private const string ApiKeyHeaderName = "X-Api-Key";
    private readonly RequestDelegate _next;
    private readonly ILogger<ApiKeyMiddleware> _logger;
    private readonly string _configuredApiKey;

    public ApiKeyMiddleware(
        RequestDelegate next,
        ILogger<ApiKeyMiddleware> logger,
        IConfiguration configuration)
    {
        _next = next;
        _logger = logger;

        _configuredApiKey = configuration["Authentication:ApiKey"]
            ?? throw new InvalidOperationException(
                "Authentication:ApiKey is not configured in appsettings.json");
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Allow health check endpoints through without authentication
        if (context.Request.Path.StartsWithSegments("/health"))
        {
            await _next(context);
            return;
        }

        if (!context.Request.Headers.TryGetValue(
                ApiKeyHeaderName, out var extractedApiKey))
        {
            _logger.LogWarning(
                "API key missing from request {Method} {Path}",
                context.Request.Method,
                context.Request.Path);

            context.Response.StatusCode = StatusCodes.Status401Unauthorized;
            await context.Response.WriteAsJsonAsync(new
            {
                Error = "API key is required",
                Detail = $"Provide a valid API key in the {ApiKeyHeaderName} header"
            });
            return; // Short-circuit -- do NOT call _next
        }

        // Use a constant-time comparison to prevent timing attacks
        if (!CryptographicOperations.FixedTimeEquals(
                Encoding.UTF8.GetBytes(_configuredApiKey),
                Encoding.UTF8.GetBytes(extractedApiKey!)))
        {
            _logger.LogWarning(
                "Invalid API key provided for {Method} {Path}",
                context.Request.Method,
                context.Request.Path);

            context.Response.StatusCode = StatusCodes.Status403Forbidden;
            await context.Response.WriteAsJsonAsync(new
            {
                Error = "Invalid API key"
            });
            return; // Short-circuit
        }

        await _next(context); // Key is valid, continue the pipeline
    }
}

public static class ApiKeyMiddlewareExtensions
{
    public static IApplicationBuilder UseApiKeyAuthentication(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<ApiKeyMiddleware>();
    }
}
```

Configuration in `appsettings.json`:

```json
{
  "Authentication": {
    "ApiKey": "your-secret-api-key-here"
  }
}
```

> [!warning] Common Misconception
> String comparison with `==` is vulnerable to **timing attacks** -- an attacker can infer the correct key character by character based on response time differences. Always use `CryptographicOperations.FixedTimeEquals()` for secret comparisons. This method compares the entire byte sequence in constant time regardless of where a mismatch occurs.

> [!tip]
> For production APIs, consider using ASP.NET Core's built-in authentication system with a custom `AuthenticationHandler<T>` instead of middleware. The built-in system integrates with `[Authorize]` attributes, policies, and claims. API key middleware is appropriate for simple internal services.

> [!summary] Section Summary
> API key middleware extracts a key from the `X-Api-Key` header, compares it using constant-time comparison against a configured value, and short-circuits with 401/403 for invalid requests. It allows whitelisted paths (like health checks) through without authentication.

### Global Exception Handling Middleware

Catches unhandled exceptions from downstream middleware and controllers, logs them, and returns a standardized **ProblemDetails** JSON response per RFC 7807.

```csharp
using System.Net;
using Microsoft.AspNetCore.Mvc;

public class GlobalExceptionHandlerMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionHandlerMiddleware> _logger;
    private readonly IHostEnvironment _environment;

    public GlobalExceptionHandlerMiddleware(
        RequestDelegate next,
        ILogger<GlobalExceptionHandlerMiddleware> logger,
        IHostEnvironment environment)
    {
        _next = next;
        _logger = logger;
        _environment = environment;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(
                ex,
                "Unhandled exception processing {Method} {Path}",
                context.Request.Method,
                context.Request.Path);

            await HandleExceptionAsync(context, ex);
        }
    }

    private async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        // Map exception types to appropriate HTTP status codes
        var (statusCode, title) = exception switch
        {
            ArgumentException =>
                (StatusCodes.Status400BadRequest, "Bad Request"),
            KeyNotFoundException =>
                (StatusCodes.Status404NotFound, "Resource Not Found"),
            UnauthorizedAccessException =>
                (StatusCodes.Status401Unauthorized, "Unauthorized"),
            InvalidOperationException =>
                (StatusCodes.Status409Conflict, "Conflict"),
            _ =>
                (StatusCodes.Status500InternalServerError, "Internal Server Error")
        };

        var problemDetails = new ProblemDetails
        {
            Status = statusCode,
            Title = title,
            Detail = _environment.IsDevelopment()
                ? exception.Message
                : "An error occurred while processing your request.",
            Instance = context.Request.Path,
            Type = $"https://httpstatuses.com/{statusCode}"
        };

        // Add trace ID for correlation with logs
        problemDetails.Extensions["traceId"] =
            context.TraceIdentifier;

        // Include stack trace only in development
        if (_environment.IsDevelopment())
        {
            problemDetails.Extensions["stackTrace"] =
                exception.StackTrace;
            problemDetails.Extensions["exceptionType"] =
                exception.GetType().FullName;
        }

        context.Response.StatusCode = statusCode;
        context.Response.ContentType = "application/problem+json";

        await context.Response.WriteAsJsonAsync(problemDetails);
    }
}

public static class GlobalExceptionHandlerMiddlewareExtensions
{
    public static IApplicationBuilder UseGlobalExceptionHandler(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<GlobalExceptionHandlerMiddleware>();
    }
}
```

> [!danger]
> Exception handling middleware must be placed **first** (or very early) in the pipeline. If it is placed after another middleware that throws, the exception will not be caught. The general rule: the exception handler wraps everything it needs to protect.

Usage:

```csharp
var app = builder.Build();

app.UseGlobalExceptionHandler(); // FIRST in the pipeline
app.UseCorrelationId();
app.UseRequestTiming();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

Example response for an unhandled `KeyNotFoundException`:

```json
{
  "type": "https://httpstatuses.com/404",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "Order with ID 12345 was not found.",
  "instance": "/api/orders/12345",
  "traceId": "0HN5LQVM8FJKP:00000001"
}
```

> [!warning] Common Misconception
> Developers sometimes check `context.Response.HasStarted` only to log a warning and then attempt to write anyway. Once `HasStarted` is `true`, you **cannot** modify the status code or headers. The response is already on the wire. Exception handling middleware works because it catches the exception before the response body is written. If your downstream middleware starts streaming a response and then throws, the exception handler cannot produce a clean ProblemDetails response.

> [!summary] Section Summary
> Global exception handling middleware wraps the entire downstream pipeline in a try-catch, maps exception types to HTTP status codes, and returns RFC 7807 ProblemDetails JSON. Include stack traces only in Development. Place this middleware first in the pipeline. Be aware that it cannot recover if the response has already started streaming.
