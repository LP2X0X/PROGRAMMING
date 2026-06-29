---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---


The most powerful pattern combines `IExceptionHandler` (.NET 8+) with ProblemDetails to convert domain exceptions into standardized API error responses. This is where [[Exception Handling]] and ProblemDetails come together.

```csharp
public class DomainExceptionHandler : IExceptionHandler
{
    private readonly IProblemDetailsService _problemDetailsService;
    private readonly ILogger<DomainExceptionHandler> _logger;

    public DomainExceptionHandler(
        IProblemDetailsService problemDetailsService,
        ILogger<DomainExceptionHandler> logger)
    {
        _problemDetailsService = problemDetailsService;
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        // Map the exception to a ProblemDetails response
        var (statusCode, type, title, errorCode) = exception switch
        {
            NotFoundException => (
                404,
                "https://example.com/errors/not-found",
                "Resource Not Found",
                "NOT_FOUND"),

            ConflictException => (
                409,
                "https://example.com/errors/conflict",
                "Resource Conflict",
                "CONFLICT"),

            ValidationException => (
                422,
                "https://example.com/errors/validation",
                "Validation Failed",
                "VALIDATION_FAILED"),

            ForbiddenException => (
                403,
                "https://example.com/errors/forbidden",
                "Access Denied",
                "ACCESS_DENIED"),

            _ => (0, "", "", "")  // Not handled by this handler
        };

        if (statusCode == 0)
            return false;  // Pass to the next handler

        _logger.LogWarning(exception,
            "Domain exception {ErrorCode} on {Method} {Path}",
            errorCode,
            httpContext.Request.Method,
            httpContext.Request.Path);

        httpContext.Response.StatusCode = statusCode;

        var problemDetails = new ProblemDetails
        {
            Type = type,
            Title = title,
            Status = statusCode,
            Detail = exception.Message,
            Instance = httpContext.Request.Path
        };

        problemDetails.Extensions["errorCode"] = errorCode;

        // Handle ValidationException specially -- include field errors
        if (exception is ValidationException validationEx)
        {
            problemDetails = new ValidationProblemDetails(validationEx.Errors)
            {
                Type = type,
                Title = title,
                Status = statusCode,
                Detail = exception.Message,
                Instance = httpContext.Request.Path
            };
            problemDetails.Extensions["errorCode"] = errorCode;
        }

        // Use IProblemDetailsService to apply global customizations
        await _problemDetailsService.WriteAsync(new ProblemDetailsContext
        {
            HttpContext = httpContext,
            ProblemDetails = problemDetails
        });

        return true;
    }
}
```

## Registration

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = ctx =>
    {
        ctx.ProblemDetails.Extensions["traceId"] = 
            ctx.HttpContext.TraceIdentifier;
    };
});

builder.Services.AddExceptionHandler<DomainExceptionHandler>();
builder.Services.AddExceptionHandler<FallbackExceptionHandler>();

var app = builder.Build();

app.UseExceptionHandler();
app.MapControllers();
app.Run();
```

> [!tip]
> Using `IProblemDetailsService` inside your `IExceptionHandler` (instead of writing JSON directly) ensures that the global `CustomizeProblemDetails` customizations are applied. If you use `WriteAsJsonAsync` directly, you bypass the global configuration and lose consistency.

> [!summary] Section Summary
> - `IExceptionHandler` combined with `IProblemDetailsService` provides the cleanest integration of [[Exception Handling]] and ProblemDetails
> - Map domain exceptions to specific `type` URIs, `title` strings, and `errorCode` extensions via pattern matching
> - Use `IProblemDetailsService.WriteAsync()` instead of direct JSON serialization to ensure global customizations are applied
> - Handle `ValidationException` specially by using `ValidationProblemDetails` with field-level errors
> - Register specific handlers before the catch-all to maintain clean separation of concerns
