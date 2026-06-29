---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---


When `UseExceptionHandler` catches an exception and re-executes the pipeline, it stores the exception details in an **`IExceptionHandlerPathFeature`** object, accessible through the `HttpContext.Features` collection.

```csharp
public interface IExceptionHandlerPathFeature : IExceptionHandlerFeature
{
    // The original request path where the exception occurred
    string? Path { get; }

    // The endpoint that was matched (if any) before the exception
    RouteValueDictionary? RouteValues { get; }
}

public interface IExceptionHandlerFeature
{
    // The actual exception that was thrown
    Exception Error { get; }

    // The endpoint that was originally matched
    Endpoint? Endpoint { get; }
}
```

Here is how to use it in practice:

```csharp
[Route("/Error")]
public class ErrorController : Controller
{
    private readonly ILogger<ErrorController> _logger;

    public ErrorController(ILogger<ErrorController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    [HttpPost]   // Handle both GET and POST since the original method is preserved
    public IActionResult HandleError()
    {
        var feature = HttpContext.Features.Get<IExceptionHandlerPathFeature>();

        if (feature is not null)
        {
            // Log the full exception with context
            _logger.LogError(feature.Error,
                "Unhandled exception on {Path} at endpoint {Endpoint}",
                feature.Path,
                feature.Endpoint?.DisplayName);

            // You can inspect the exception type to customize the response
            var statusCode = feature.Error switch
            {
                FileNotFoundException => 404,
                UnauthorizedAccessException => 403,
                _ => 500
            };

            return View("Error", new ErrorViewModel
            {
                StatusCode = statusCode,
                RequestId = HttpContext.TraceIdentifier,
                // NEVER expose feature.Error.Message to the user in production
                Message = statusCode == 404
                    ? "The requested resource was not found."
                    : "An unexpected error occurred."
            });
        }

        return View("Error", new ErrorViewModel
        {
            StatusCode = 500,
            RequestId = HttpContext.TraceIdentifier,
            Message = "An unexpected error occurred."
        });
    }
}
```

> [!tip]
> Always check for `null` when accessing `IExceptionHandlerPathFeature`. It will be `null` if the error page is accessed directly (e.g., someone navigates to `/Error` manually) rather than through the exception handler re-execution.

> [!summary] Section Summary
> - `IExceptionHandlerPathFeature` provides the original exception (`Error`), the request path (`Path`), and the matched endpoint (`Endpoint`)
> - Access it via `HttpContext.Features.Get<IExceptionHandlerPathFeature>()`
> - Always null-check -- the feature is only present when the error handler was invoked by the exception middleware
> - Use it to log full exception details and to customize the error response based on exception type
