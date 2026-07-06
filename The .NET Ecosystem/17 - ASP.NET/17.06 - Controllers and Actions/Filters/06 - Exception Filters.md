---
tags:
  - csharp
  - asp-net-core
  - filters
  - controllers
  - cross-cutting
---


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
