---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---


Here is a complete error handling configuration for an application that serves both MVC views and API endpoints, using the .NET 8+ `IExceptionHandler` approach combined with [[Problem Details]] and [[ILogger and Logging|structured logging]].

## Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddControllersWithViews();
builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = ctx =>
    {
        ctx.ProblemDetails.Extensions["traceId"] = ctx.HttpContext.TraceIdentifier;
        ctx.ProblemDetails.Extensions["nodeId"] = Environment.MachineName;
    };
});

// Register exception handlers in priority order
builder.Services.AddExceptionHandler<ValidationExceptionHandler>();
builder.Services.AddExceptionHandler<DomainExceptionHandler>();
builder.Services.AddExceptionHandler<FallbackExceptionHandler>();

var app = builder.Build();

// Pipeline configuration
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    // Exception handler for unhandled exceptions
    app.UseExceptionHandler();

    // Status code pages for non-exception errors (404, 403, etc.)
    app.UseStatusCodePagesWithReExecute("/errors/{0}");

    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.MapControllers();  // For [ApiController] attributed controllers

app.Run();
```

## ValidationExceptionHandler.cs

```csharp
public class ValidationExceptionHandler : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        if (exception is not ValidationException validationEx)
            return false;

        httpContext.Response.StatusCode = 422;

        await httpContext.Response.WriteAsJsonAsync(new ValidationProblemDetails(
            validationEx.Errors)
        {
            Type = "https://httpstatuses.com/422",
            Title = "Validation Failed",
            Status = 422,
            Detail = "One or more validation errors occurred.",
            Instance = httpContext.Request.Path,
            Extensions =
            {
                ["traceId"] = httpContext.TraceIdentifier,
                ["errorCode"] = "VALIDATION_FAILED"
            }
        }, cancellationToken);

        return true;
    }
}
```

## ErrorsController.cs (for Status Code Pages)

```csharp
public class ErrorsController : Controller
{
    [Route("/errors/{statusCode}")]
    [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None,
        NoStore = true)]
    public IActionResult HandleStatusCode(int statusCode)
    {
        // API requests get JSON ProblemDetails even for status code pages
        if (Request.Headers.Accept.ToString().Contains("application/json")
            || Request.Path.Value?.StartsWith("/api") == true)
        {
            return new ObjectResult(new ProblemDetails
            {
                Type = $"https://httpstatuses.com/{statusCode}",
                Title = ReasonPhrases.GetReasonPhrase(statusCode),
                Status = statusCode,
                Instance = HttpContext.Features
                    .Get<IStatusCodeReExecuteFeature>()?.OriginalPath
            })
            {
                StatusCode = statusCode
            };
        }

        // MVC requests get a view
        ViewData["StatusCode"] = statusCode;
        return statusCode switch
        {
            404 => View("NotFound"),
            403 => View("Forbidden"),
            _ => View("GenericError")
        };
    }
}
```

> [!summary] Section Summary
> - A complete setup combines `IExceptionHandler` (for exceptions), status code pages (for non-exception errors), and the Developer Exception Page (for development only)
> - Register exception handlers in priority order: most specific first, catch-all last
> - Use `AddProblemDetails()` to configure [[Problem Details]] globally with trace IDs and custom extensions
> - The errors controller handles both JSON and HTML responses based on the request type
> - In development, `UseDeveloperExceptionPage()` overrides everything for rich debugging; in production, the layered approach provides safe, consistent error responses
