---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---


The built-in `UseExceptionHandler` works well for simple scenarios, but real-world applications often need more control -- especially when serving both MVC views and API endpoints from the same application. A custom exception handling middleware gives you complete control over the error response format.

```csharp
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionMiddleware> _logger;
    private readonly IHostEnvironment _environment;

    public GlobalExceptionMiddleware(
        RequestDelegate next,
        ILogger<GlobalExceptionMiddleware> logger,
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
            _logger.LogError(ex,
                "Unhandled exception for {Method} {Path}",
                context.Request.Method,
                context.Request.Path);

            // Do not modify the response if it has already started sending
            if (context.Response.HasStarted)
            {
                _logger.LogWarning(
                    "Response has already started, cannot modify error response");
                throw;  // Re-throw -- nothing else we can do
            }

            context.Response.Clear();

            if (IsApiRequest(context))
            {
                await HandleApiException(context, ex);
            }
            else
            {
                await HandleMvcException(context, ex);
            }
        }
    }

    private static bool IsApiRequest(HttpContext context)
    {
        // Check if the request targets an API endpoint
        return context.Request.Path.StartsWithSegments("/api")
            || context.Request.Headers.Accept
                   .ToString().Contains("application/json");
    }

    private async Task HandleApiException(HttpContext context, Exception ex)
    {
        var (statusCode, title) = MapExceptionToStatusCode(ex);

        context.Response.StatusCode = statusCode;
        context.Response.ContentType = "application/problem+json";

        var problemDetails = new
        {
            type = $"https://httpstatuses.com/{statusCode}",
            title,
            status = statusCode,
            detail = _environment.IsDevelopment()
                ? ex.Message
                : "An error occurred while processing your request.",
            instance = context.Request.Path.ToString(),
            traceId = context.TraceIdentifier
        };

        await context.Response.WriteAsJsonAsync(problemDetails);
    }

    private async Task HandleMvcException(HttpContext context, Exception ex)
    {
        var (statusCode, _) = MapExceptionToStatusCode(ex);
        context.Response.StatusCode = statusCode;

        // Re-execute to the error page
        context.Request.Path = "/Home/Error";
        context.Features.Set<IExceptionHandlerPathFeature>(
            new ExceptionHandlerFeature
            {
                Error = ex,
                Path = context.Request.Path
            });

        // For MVC, you might redirect to an error view or use a simple HTML response
        context.Response.ContentType = "text/html";
        await context.Response.WriteAsync(
            "<html><body><h1>An error occurred</h1>" +
            "<p>Please try again later.</p></body></html>");
    }

    private static (int StatusCode, string Title) MapExceptionToStatusCode(
        Exception ex)
    {
        return ex switch
        {
            ArgumentException => (400, "Bad Request"),
            KeyNotFoundException => (404, "Not Found"),
            UnauthorizedAccessException => (401, "Unauthorized"),
            InvalidOperationException => (409, "Conflict"),
            NotImplementedException => (501, "Not Implemented"),
            TimeoutException => (504, "Gateway Timeout"),
            _ => (500, "Internal Server Error")
        };
    }
}

// Extension method for clean registration
public static class GlobalExceptionMiddlewareExtensions
{
    public static IApplicationBuilder UseGlobalExceptionHandler(
        this IApplicationBuilder app)
    {
        return app.UseMiddleware<GlobalExceptionMiddleware>();
    }
}
```

Registration in `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Must be the outermost middleware
app.UseGlobalExceptionHandler();

app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

> [!warning] Common Misconception
> A common mistake is checking `context.Response.HasStarted` and then trying to write a custom error response anyway. Once the response headers have been sent to the client (because the endpoint started writing the body), you **cannot** change the status code, headers, or clear the response. You must re-throw the exception and let the server handle the connection reset. This typically happens with streaming responses or when the response body is very large.

> [!summary] Section Summary
> - Custom exception middleware gives full control over error responses for both API and MVC requests
> - Check `context.Response.HasStarted` before attempting to modify the response
> - Use content negotiation (path prefix, Accept header) to return JSON [[Problem Details]] for APIs and HTML for MVC
> - Map domain exception types to appropriate HTTP status codes using pattern matching
> - Always register the exception middleware as the outermost middleware in the pipeline
