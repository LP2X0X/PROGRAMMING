---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---


A clean pattern for production applications is to define custom domain exception types and map them to appropriate HTTP status codes in your exception handling middleware.

## Define Domain Exceptions

```csharp
// Base class for domain exceptions that carry an HTTP status code
public abstract class DomainException : Exception
{
    public abstract int StatusCode { get; }
    public abstract string ErrorCode { get; }

    protected DomainException(string message) : base(message) { }
    protected DomainException(string message, Exception inner) : base(message, inner) { }
}

public class NotFoundException : DomainException
{
    public override int StatusCode => 404;
    public override string ErrorCode => "RESOURCE_NOT_FOUND";

    public NotFoundException(string resource, object id)
        : base($"{resource} with identifier '{id}' was not found.") { }
}

public class ConflictException : DomainException
{
    public override int StatusCode => 409;
    public override string ErrorCode => "RESOURCE_CONFLICT";

    public ConflictException(string message) : base(message) { }
}

public class ValidationException : DomainException
{
    public override int StatusCode => 422;
    public override string ErrorCode => "VALIDATION_FAILED";
    public IDictionary<string, string[]> Errors { get; }

    public ValidationException(IDictionary<string, string[]> errors)
        : base("One or more validation errors occurred.")
    {
        Errors = errors;
    }
}

public class ForbiddenException : DomainException
{
    public override int StatusCode => 403;
    public override string ErrorCode => "ACCESS_DENIED";

    public ForbiddenException(string message) : base(message) { }
}
```

## Map in Middleware

```csharp
catch (Exception ex)
{
    var (statusCode, errorCode, detail) = ex switch
    {
        DomainException domainEx =>
            (domainEx.StatusCode, domainEx.ErrorCode, domainEx.Message),

        ArgumentException argEx =>
            (400, "BAD_REQUEST", argEx.Message),

        OperationCanceledException =>
            (499, "CLIENT_CLOSED_REQUEST", "The client cancelled the request."),

        _ => (500, "INTERNAL_ERROR",
              "An unexpected error occurred. Please try again later.")
    };

    // Only log 500s as errors; 4xx are expected and logged as warnings
    if (statusCode >= 500)
        _logger.LogError(ex, "Server error on {Path}", context.Request.Path);
    else
        _logger.LogWarning(ex, "Client error {StatusCode} on {Path}",
            statusCode, context.Request.Path);

    context.Response.StatusCode = statusCode;
    context.Response.ContentType = "application/problem+json";

    await context.Response.WriteAsJsonAsync(new
    {
        type = $"https://httpstatuses.com/{statusCode}",
        title = ReasonPhrases.GetReasonPhrase(statusCode),
        status = statusCode,
        detail,
        errorCode,
        instance = context.Request.Path.ToString(),
        traceId = context.TraceIdentifier
    });
}
```

> [!tip]
> Log client errors (4xx) as **Warning**, not **Error**. A 404 is not a bug in your application -- it is expected behavior. Reserve the Error level for genuine server-side failures (5xx). This keeps your error alerts meaningful and avoids alert fatigue.

> [!summary] Section Summary
> - Define domain exception types that carry their own HTTP status code and error code
> - Use pattern matching in middleware to map exception types to appropriate responses
> - Distinguish between client errors (4xx, log as Warning) and server errors (5xx, log as Error)
> - Include machine-readable error codes alongside HTTP status codes for programmatic client handling
> - Follow the [[Problem Details]] format for API error responses
