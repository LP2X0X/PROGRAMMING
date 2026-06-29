---
tags: [csharp, asp-net-core, error-handling, exceptions, middleware]
aliases: [ASP.NET Core Exception Handling, Global Error Handling, Exception Middleware]
status: complete
date: 2026-06-18
---

# Exception Handling

Every web application will eventually encounter unexpected errors -- database timeouts, null references, failed external API calls, file system issues. How you handle these errors determines whether your users see a helpful message or a raw stack trace, and whether your operations team can diagnose problems or is left guessing.

In ASP.NET Core, exception handling is built around the **middleware pipeline**. The framework provides built-in middleware for development and production error handling, and gives you extension points to build custom solutions that handle both MVC views and API JSON responses from a single codebase.

---

## Table of Contents

- [[#Why Exception Handling Matters]]
- [[#Developer Exception Page]]
- [[#Production Exception Handler]]
- [[#IExceptionHandlerPathFeature -- Accessing Exception Details]]
- [[#Custom Global Exception Handling Middleware]]
- [[#Exception Filters vs Exception Middleware]]
- [[#Status Code Pages]]
- [[#Mapping Exception Types to HTTP Status Codes]]
- [[#IExceptionHandler Interface (.NET 8+)]]
- [[#try-catch in Controllers -- When and When Not To]]
- [[#Never Expose Internal Details in Production]]
- [[#Real-World -- Complete Error Handling Setup]]
- [[#Related Topics]]
- [[#Further Reading]]
- [[#Comprehensive Summary]]

---

## Why Exception Handling Matters

When an unhandled exception escapes your application code and reaches the ASP.NET Core hosting layer, the default behavior is to return a **blank 500 Internal Server Error** response with no body. This is terrible for everyone:

- **Users** see a cryptic error or a blank page with no guidance
- **Developers** have no diagnostic information in the response (unless they check server logs)
- **Attackers** may receive stack traces, connection strings, or internal paths if the Developer Exception Page is accidentally enabled in production

The consequences of poor exception handling are real:

1. **Information leakage** -- stack traces reveal internal implementation details, file paths, assembly names, database connection strings, and even query parameters. This is a security vulnerability.
2. **Poor user experience** -- users who see a raw 500 error have no idea what happened or what to do next
3. **Difficult debugging** -- without structured logging of exceptions, diagnosing production issues becomes guesswork
4. **Inconsistent responses** -- APIs that sometimes return JSON and sometimes return HTML error pages break client applications

> [!danger]
> An unhandled exception in production without proper error handling middleware can expose your application's internals. The default Kestrel response for an unhandled exception is a 500 status with an empty body, but if the Developer Exception Page is accidentally left on, the full stack trace, source code snippets, request headers, cookies, and routing data are all visible to anyone making the request.

> [!summary] Section Summary
> - Unhandled exceptions produce blank 500 responses by default, or leak internal details if the Developer Exception Page is on in production
> - Poor error handling leads to security vulnerabilities, bad UX, difficult debugging, and inconsistent API responses
> - Proper exception handling middleware is not optional -- it is a fundamental requirement for any production application

---

## Developer Exception Page

The **Developer Exception Page** is a built-in middleware that provides rich, detailed error information during development. It is enabled by default in the Development environment in .NET 6+ project templates.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// In .NET 6+, this is automatically added when
// app.Environment.IsDevelopment() is true.
// You can also add it explicitly:
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
```

When an unhandled exception occurs, this middleware catches it and renders a detailed HTML page that includes:

| Information | Description |
|---|---|
| **Stack trace** | Full call stack with source file paths and line numbers |
| **Source code** | The actual source code lines around where the exception occurred |
| **Request details** | HTTP method, URL, query string, headers |
| **Cookies** | All cookies sent with the request |
| **Route data** | Route values, endpoint metadata |
| **Exception details** | Exception type, message, inner exceptions |

> [!danger]
> The Developer Exception Page must **never** be enabled in production. It exposes source code, file paths, configuration details, and potentially sensitive data. The `IsDevelopment()` check is critical. If your environment variable `ASPNETCORE_ENVIRONMENT` is not set, the default is `Production`, which is the safe default.

> [!ad-note]
> In .NET 6 and later with `WebApplication.CreateBuilder()`, the Developer Exception Page is added automatically when the environment is Development. You do not need to call `app.UseDeveloperExceptionPage()` explicitly unless you are using `WebApplication.CreateSlimBuilder()` or a custom hosting setup.

> [!summary] Section Summary
> - The Developer Exception Page provides rich error details including stack traces, source code, request info, and cookies
> - It is automatically enabled in the Development environment in .NET 6+ templates
> - It must never be active in production due to the sensitive information it exposes
> - The `ASPNETCORE_ENVIRONMENT` variable controls which environment is active

---

## Production Exception Handler

For production, ASP.NET Core provides `UseExceptionHandler()`, which catches exceptions from downstream middleware, logs them, and re-executes the pipeline to a specified error-handling path.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    // Catches any unhandled exception from downstream middleware,
    // then re-executes the pipeline targeting "/Home/Error"
    app.UseExceptionHandler("/Home/Error");
}

app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();
app.Run();
```

### How UseExceptionHandler Works Internally

Understanding the internal mechanism is essential for debugging. Here is what happens step by step:

1. An exception is thrown somewhere in the pipeline (controller, service, another middleware)
2. The exception propagates up the middleware chain until it reaches `UseExceptionHandler`
3. `UseExceptionHandler` **catches** the exception and **clears** the response (status code, headers, body)
4. It sets the response status code to **500**
5. It stores the exception details in `IExceptionHandlerPathFeature` on the `HttpContext.Features` collection
6. It **re-executes** the middleware pipeline from the exception handler middleware onward, but with `HttpContext.Request.Path` set to the error path (e.g., `/Home/Error`)
7. The re-executed pipeline hits the error controller/endpoint, which renders the error page
8. If the error handler itself throws, the middleware catches that too and returns a plain-text response

> [!ad-note] Why Re-Execute Instead of Writing the Error Directly?
> When your code throws, the response may already be ==partially written== — some headers may have been sent, some data may have been flushed to the response body. At that point, the response is in a ==corrupted state== and you cannot reliably write a clean JSON or HTML error body on top of it. By **clearing the response** (step 3) and **re-executing the pipeline with a new path** (step 6), the exception handler creates a ==clean slate==. The second pass goes through the full pipeline — DI, serialization, content negotiation — and produces a proper, well-formatted error response as if it were a normal request.

```csharp
// The error controller that handles re-executed requests
public class HomeController : Controller
{
    [Route("/Home/Error")]
    [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
    public IActionResult Error()
    {
        // Access the original exception details
        var exceptionFeature = HttpContext.Features
            .Get<IExceptionHandlerPathFeature>();

        // Log or inspect the exception
        var originalException = exceptionFeature?.Error;
        var originalPath = exceptionFeature?.Path;

        return View(new ErrorViewModel
        {
            RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier,
            Message = "An unexpected error occurred. Please try again later."
        });
    }
}
```

> [!warning] Common Misconception
> `UseExceptionHandler` does **not** redirect the browser to the error path. It re-executes the pipeline internally on the same request. The client's URL does not change, there is no 302 redirect, and the original HTTP method is preserved. This means if a POST request fails, the error handler receives the context of a POST request, not a GET.

There is also an overload that accepts a lambda for inline error handling without a separate error path:

```csharp
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        context.Response.StatusCode = 500;
        context.Response.ContentType = "text/html";

        var exceptionFeature = context.Features
            .Get<IExceptionHandlerPathFeature>();

        await context.Response.WriteAsync("<h1>An error occurred</h1>");
        // Do NOT write exception details in production
    });
});
```

> [!summary] Section Summary
> - `UseExceptionHandler` catches unhandled exceptions and re-executes the pipeline to a specified error path
> - The re-execution is internal -- no browser redirect, no URL change, same HTTP method
> - The original exception is accessible via `IExceptionHandlerPathFeature` on `HttpContext.Features`
> - Always place `UseExceptionHandler` as the outermost middleware to catch all downstream exceptions
> - A lambda overload allows inline error handling without a separate controller action

---

## IExceptionHandlerPathFeature -- Accessing Exception Details

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

---

## Custom Global Exception Handling Middleware

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

---

## Exception Filters vs Exception Middleware

ASP.NET Core provides two mechanisms for handling exceptions: **exception filters** (part of MVC) and **exception middleware** (part of the pipeline). They serve different purposes and have different scopes.

### Exception Filters

Exception filters implement `IExceptionFilter` or `IAsyncExceptionFilter` and are part of the **MVC filter pipeline**. They only catch exceptions that occur within MVC action execution.

```csharp
public class CustomExceptionFilter : IExceptionFilter
{
    private readonly ILogger<CustomExceptionFilter> _logger;

    public CustomExceptionFilter(ILogger<CustomExceptionFilter> logger)
    {
        _logger = logger;
    }

    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception,
            "Exception in {Controller}.{Action}",
            context.RouteData.Values["controller"],
            context.RouteData.Values["action"]);

        context.Result = new ObjectResult(new
        {
            error = "An error occurred",
            traceId = context.HttpContext.TraceIdentifier
        })
        {
            StatusCode = 500
        };

        // Mark the exception as handled -- stops propagation
        context.ExceptionHandled = true;
    }
}

// Registration -- globally or per controller/action
builder.Services.AddControllers(options =>
{
    options.Filters.Add<CustomExceptionFilter>();
});
```

### Comparison Table

| Aspect | Exception Filters | Exception Middleware |
|---|---|---|
| **Scope** | MVC actions and Razor Pages only | Entire request pipeline |
| **Catches from** | Controller actions, action filters, result filters | Any middleware, controllers, Razor Pages, minimal APIs |
| **Registration** | `options.Filters.Add<>()` or `[TypeFilter]` attribute | `app.UseMiddleware<>()` or `app.UseExceptionHandler()` |
| **DI support** | Yes, via `[TypeFilter]` or `[ServiceFilter]` | Yes, via constructor injection |
| **Access to MVC context** | Yes -- `ExceptionContext` with `ActionDescriptor`, `RouteData`, etc. | No -- only `HttpContext` |
| **Can set `IActionResult`** | Yes -- `context.Result = new ObjectResult(...)` | No -- must write directly to `HttpResponse` |
| **Order of execution** | Runs within the MVC pipeline (after model binding, before result execution) | Runs at the middleware level (outermost layer) |

> [!tip]
> Use exception filters when you need MVC-specific context (which controller, which action, route data) and the exception originates from controller logic. Use exception middleware for everything else -- including exceptions from other middleware, minimal API endpoints, and as a global safety net. In most applications, **use both**: middleware as the outermost catch-all, and filters for MVC-specific error formatting.

> [!summary] Section Summary
> - Exception filters are MVC-specific and catch exceptions from controller actions, with access to `ActionDescriptor`, `RouteData`, and the ability to set `IActionResult`
> - Exception middleware operates at the pipeline level and catches exceptions from all sources
> - Filters cannot catch exceptions from middleware, minimal APIs, or non-MVC code
> - Best practice: use middleware as the global safety net and filters for MVC-specific formatting

---

## Status Code Pages

Not all errors involve exceptions. A 404 (Not Found), 403 (Forbidden), or other non-success status code might be set by a controller or routing middleware without throwing an exception. The **Status Code Pages** middleware handles these non-exception error status codes.

### UseStatusCodePages

The simplest form provides a plain-text response for any non-success, non-redirect status code:

```csharp
app.UseStatusCodePages();
// Returns: "Status Code: 404; Not Found"
```

You can customize the response with a format string:

```csharp
app.UseStatusCodePages("text/plain", "Error: status code {0}");
```

### UseStatusCodePagesWithReExecute

This is the most useful variant. It re-executes the pipeline to a specified path, allowing you to render a full error page with your layout:

```csharp
// The {0} placeholder is replaced with the status code
app.UseStatusCodePagesWithReExecute("/errors/{0}");
```

```csharp
public class ErrorsController : Controller
{
    [Route("/errors/{statusCode}")]
    public IActionResult HandleStatusCode(int statusCode)
    {
        return statusCode switch
        {
            404 => View("NotFound"),   // ~/Views/Errors/NotFound.cshtml
            403 => View("Forbidden"),  // ~/Views/Errors/Forbidden.cshtml
            _ => View("GenericError", new ErrorViewModel { StatusCode = statusCode })
        };
    }
}
```

### UseStatusCodePagesWithRedirects

This variant issues a **302 redirect** to the error page. Generally **avoid** this because:

- The browser URL changes to `/errors/404`, which is misleading
- The original URL is lost
- The status code sent to the browser is 302, then 200 (not the original 404) -- bad for SEO and client-side error handling

```csharp
// Generally avoid this -- prefer ReExecute
app.UseStatusCodePagesWithRedirects("/errors/{0}");
```

### Important Placement

Status code pages middleware must be placed **early** in the pipeline, but **after** the exception handler:

```csharp
app.UseExceptionHandler("/Home/Error");  // Catches exceptions
app.UseStatusCodePages();                 // Catches non-exception error status codes
app.UseStaticFiles();
app.UseRouting();
```

> [!warning] Common Misconception
> Status code pages middleware does **not** handle exceptions. It handles status codes set by downstream middleware and endpoints when no response body has been written. If a controller returns `NotFound()` (which sets 404 but writes no body), the status code pages middleware generates the error response. But if a controller throws a `KeyNotFoundException`, that is caught by the exception handler, not the status code pages middleware.

> [!ad-note]
> `UseStatusCodePagesWithReExecute` preserves the original URL in the browser address bar and returns the correct status code to the client (e.g., 404). This is the preferred approach for SEO and proper client-side error handling. The re-execute is internal, just like `UseExceptionHandler`.

> [!summary] Section Summary
> - Status code pages middleware handles non-exception error responses (404, 403, etc.) where no response body was written
> - `UseStatusCodePagesWithReExecute("/errors/{0}")` is the preferred variant -- it preserves the URL and returns the correct status code
> - Avoid `UseStatusCodePagesWithRedirects` because it changes the URL and loses the original status code
> - Place status code pages middleware after exception handling but before routing
> - This middleware complements, not replaces, exception handling middleware

---

## Mapping Exception Types to HTTP Status Codes

A clean pattern for production applications is to define custom domain exception types and map them to appropriate HTTP status codes in your exception handling middleware.

### Define Domain Exceptions

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

### Map in Middleware

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

---

## IExceptionHandler Interface (.NET 8+)

.NET 8 introduced the **`IExceptionHandler`** interface as the modern, DI-friendly way to handle exceptions. It replaces the need for custom exception middleware in many scenarios and supports multiple handlers in priority order.

```csharp
public interface IExceptionHandler
{
    ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken);
}
```

The handler returns `true` if it handled the exception (stopping the chain) or `false` to pass it to the next handler.

### Implementing IExceptionHandler

```csharp
// Handler for domain-specific exceptions
public class DomainExceptionHandler : IExceptionHandler
{
    private readonly ILogger<DomainExceptionHandler> _logger;

    public DomainExceptionHandler(ILogger<DomainExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        if (exception is not DomainException domainException)
            return false;  // Not our exception type -- pass to next handler

        _logger.LogWarning(exception,
            "Domain exception: {ErrorCode} on {Path}",
            domainException.ErrorCode,
            httpContext.Request.Path);

        httpContext.Response.StatusCode = domainException.StatusCode;

        await httpContext.Response.WriteAsJsonAsync(new ProblemDetails
        {
            Type = $"https://httpstatuses.com/{domainException.StatusCode}",
            Title = ReasonPhrases.GetReasonPhrase(domainException.StatusCode),
            Status = domainException.StatusCode,
            Detail = domainException.Message,
            Instance = httpContext.Request.Path,
            Extensions =
            {
                ["errorCode"] = domainException.ErrorCode,
                ["traceId"] = httpContext.TraceIdentifier
            }
        }, cancellationToken);

        return true;  // Handled -- stop the chain
    }
}

// Catch-all handler for unexpected exceptions
public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;
    private readonly IHostEnvironment _environment;

    public GlobalExceptionHandler(
        ILogger<GlobalExceptionHandler> logger,
        IHostEnvironment environment)
    {
        _logger = logger;
        _environment = environment;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception,
            "Unhandled exception on {Method} {Path}",
            httpContext.Request.Method,
            httpContext.Request.Path);

        httpContext.Response.StatusCode = 500;

        await httpContext.Response.WriteAsJsonAsync(new ProblemDetails
        {
            Type = "https://httpstatuses.com/500",
            Title = "Internal Server Error",
            Status = 500,
            Detail = _environment.IsDevelopment()
                ? exception.Message
                : "An unexpected error occurred.",
            Instance = httpContext.Request.Path,
            Extensions =
            {
                ["traceId"] = httpContext.TraceIdentifier
            }
        }, cancellationToken);

        return true;
    }
}
```

### Registration -- Order Matters

Handlers are tried **in registration order**. The first handler that returns `true` stops the chain.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register handlers in priority order -- most specific first
builder.Services.AddExceptionHandler<DomainExceptionHandler>();
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();

// Still need to add the exception handler middleware
builder.Services.AddProblemDetails();

var app = builder.Build();

// UseExceptionHandler() now uses the registered IExceptionHandler implementations
app.UseExceptionHandler();

app.MapControllers();
app.Run();
```

### Advantages Over Custom Middleware

| Aspect | Custom Middleware | IExceptionHandler (.NET 8+) |
|---|---|---|
| **DI support** | Constructor injection only (singleton lifetime) | Full DI with scoped services |
| **Multiple handlers** | Single middleware class handles all cases | Chain of handlers, each with single responsibility |
| **Testability** | Must test through the middleware pipeline | Can unit test each handler independently |
| **Registration** | `app.UseMiddleware<>()` | `builder.Services.AddExceptionHandler<>()` |
| **Integration** | Standalone | Works with built-in `UseExceptionHandler` and [[Problem Details]] service |

> [!ad-note]
> Even with `IExceptionHandler`, you still need to call `app.UseExceptionHandler()` to install the exception handling middleware. The `IExceptionHandler` implementations are called *by* that middleware when it catches an exception. Without `UseExceptionHandler()`, the handlers are never invoked.

> [!summary] Section Summary
> - `IExceptionHandler` (.NET 8+) is the modern approach to exception handling with full DI support
> - Multiple handlers are registered and tried in order -- the first to return `true` stops the chain
> - Most specific handlers (domain exceptions) should be registered before the catch-all handler
> - Each handler has a single responsibility and can be unit tested independently
> - You still need `app.UseExceptionHandler()` in the pipeline -- `IExceptionHandler` implementations run within that middleware

---

## try-catch in Controllers -- When and When Not To

A common question is whether you should use `try-catch` blocks inside controller actions. The short answer: **rarely**.

### When NOT to Use try-catch in Controllers

For most exceptions, let them propagate to the global exception handling middleware. Catching exceptions in every controller leads to duplicated error handling code and inconsistent responses.

```csharp
// BAD: Duplicated error handling in every action
[HttpGet("{id}")]
public async Task<IActionResult> GetProduct(int id)
{
    try
    {
        var product = await _productService.GetByIdAsync(id);
        return Ok(product);
    }
    catch (NotFoundException ex)
    {
        return NotFound(new { error = ex.Message });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error getting product {Id}", id);
        return StatusCode(500, new { error = "Internal error" });
    }
}

// GOOD: Let the global middleware handle it
[HttpGet("{id}")]
public async Task<IActionResult> GetProduct(int id)
{
    var product = await _productService.GetByIdAsync(id);
    return Ok(product);
    // If GetByIdAsync throws NotFoundException, the middleware maps it to 404
    // If it throws anything else, the middleware logs it and returns 500
}
```

### When try-catch IS Appropriate

Use `try-catch` in controllers when you need to **recover** from the error and continue with an alternative path, not just translate it into a different error format:

```csharp
[HttpPost("import")]
public async Task<IActionResult> ImportProducts(IFormFile file)
{
    var results = new List<ImportResult>();

    foreach (var row in ParseCsv(file))
    {
        try
        {
            // Try to import each row individually
            await _productService.CreateAsync(row);
            results.Add(new ImportResult(row.Name, Success: true));
        }
        catch (ValidationException ex)
        {
            // One bad row should not abort the entire import
            results.Add(new ImportResult(row.Name, Success: false,
                Errors: ex.Errors));
        }
        // Let other exceptions (DB down, etc.) propagate to middleware
    }

    return Ok(new { imported = results.Count(r => r.Success), results });
}
```

```csharp
// Another valid case: calling an external service with a fallback
[HttpGet("{id}")]
public async Task<IActionResult> GetProductWithReviews(int id)
{
    var product = await _productService.GetByIdAsync(id);

    try
    {
        // External review service might be down
        product.Reviews = await _reviewService.GetForProductAsync(id);
    }
    catch (HttpRequestException)
    {
        // Degrade gracefully -- return the product without reviews
        product.Reviews = Array.Empty<Review>();
        _logger.LogWarning("Review service unavailable for product {Id}", id);
    }

    return Ok(product);
}
```

> [!tip]
> **Rule of thumb:** If the catch block translates the exception into an error response, that logic belongs in middleware or an `IExceptionHandler`. If the catch block provides a **fallback behavior** and the request still succeeds, it belongs in the controller.

> [!summary] Section Summary
> - Avoid try-catch in controllers when the goal is just to format error responses -- that is the middleware's job
> - Use try-catch when you need to recover and continue (partial imports, graceful degradation with fallbacks)
> - Letting exceptions propagate to global handlers keeps controller code clean and error handling consistent
> - The exception type should carry enough information for the middleware to generate the right response

---

## Never Expose Internal Details in Production

This principle is important enough to have its own section. **Never** return exception messages, stack traces, inner exceptions, or implementation details in production error responses.

### What Gets Exposed Accidentally

```csharp
// DANGEROUS: Exposing the raw exception message
catch (Exception ex)
{
    return StatusCode(500, new { error = ex.Message });
}
// ex.Message might be:
// "Invalid column name 'Proce'. (SqlException)"
// "Access denied for user 'appuser'@'10.0.1.42' to database 'products'"
// "Object reference not set to an instance of an object."
```

These messages reveal:
- Database column names and schema
- Internal IP addresses and usernames
- Database technology (SQL Server, MySQL, etc.)
- That you have null reference bugs (code quality signal)
- File system paths in `FileNotFoundException`

### The Safe Pattern

```csharp
// SAFE: Generic message for production, detailed for development
var detail = _environment.IsDevelopment()
    ? ex.ToString()    // Full exception with stack trace for dev
    : "An unexpected error occurred. Please contact support with " +
      $"reference ID: {context.TraceIdentifier}";

// The traceId lets support find the full exception in server logs
// without exposing internals to the client
```

> [!danger]
> Even seemingly harmless exception messages can reveal too much. `"The ConnectionString property has not been initialized"` tells an attacker you use ADO.NET and have a configuration issue. `"No such host is known: payments.internal.corp.com"` reveals your internal service naming convention. Always use generic messages in production and log the full details server-side.

> [!summary] Section Summary
> - Never return `ex.Message`, `ex.StackTrace`, or `ex.ToString()` in production responses
> - Use a trace ID / correlation ID so users can reference the error when contacting support
> - Log the full exception details server-side where only your team can see them
> - Even "harmless" messages can leak database schema, internal hostnames, usernames, and technology choices

---

## Real-World -- Complete Error Handling Setup

Here is a complete error handling configuration for an application that serves both MVC views and API endpoints, using the .NET 8+ `IExceptionHandler` approach combined with [[Problem Details]] and [[ILogger and Logging|structured logging]].

### Program.cs

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

### ValidationExceptionHandler.cs

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

### ErrorsController.cs (for Status Code Pages)

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

---

## Related Topics

- [[Middleware Overview]] -- how middleware works and why exception handling must be the outermost layer
- [[ILogger and Logging]] -- structured logging for exception details and diagnostics
- [[Problem Details]] -- the RFC 9457 standard format for API error responses
- [[Filters vs Middleware]] -- deeper comparison of MVC filters and middleware
- [[Authentication and Authorization]] -- exception handling for auth-related errors

---

## Further Reading

- [[Custom Middleware]] -- writing class-based middleware with constructor injection
- [[Health Checks]] -- monitoring application health alongside error handling
- [[API Versioning]] -- consistent error responses across API versions

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Exception handling in ASP.NET Core** operates at the middleware pipeline level. The framework provides built-in middleware for two primary scenarios: the **Developer Exception Page** for development (rich stack traces, source code, request details) and **`UseExceptionHandler`** for production (catches exceptions, re-executes the pipeline to an error path without browser redirects).
>
> For non-exception error status codes (404, 403), **Status Code Pages** middleware fills the gap -- `UseStatusCodePagesWithReExecute` is preferred over redirects because it preserves the original URL and returns the correct status code.
>
> **Custom exception handling middleware** provides full control when applications serve both MVC views and API endpoints, using content negotiation to return HTML error pages or JSON [[Problem Details]] as appropriate.
>
> **.NET 8 introduced `IExceptionHandler`**, the modern approach that supports multiple handlers in priority order with full DI support. Handlers are tried sequentially -- the first to return `true` stops the chain. Register specific handlers (validation, domain exceptions) before the catch-all handler.
>
> **Exception filters** (`IExceptionFilter`) are MVC-specific and only catch exceptions from controller actions. They complement but do not replace exception middleware, which catches everything including exceptions from other middleware and minimal APIs.
>
> **Domain exception types** (NotFoundException, ValidationException, etc.) should carry their own HTTP status code and error code, enabling clean pattern matching in the exception handler. Log 4xx as Warning and 5xx as Error to avoid alert fatigue.
>
> The cardinal rule: **never expose internal details in production**. Return generic messages with a trace ID that maps to the full exception in your server logs. The Developer Exception Page must only run in the Development environment.
>
> **Controller-level try-catch** should be reserved for graceful degradation and recovery scenarios -- not for translating exceptions into error responses, which is the global handler's responsibility.