---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---


Every HTTP request should generate at least one log entry with the method, path, status code, and response time. Instead of building this yourself, use **Serilog's request logging middleware**.

## The Problem with Default Framework Logging

By default, ASP.NET Core logs **multiple** entries per request at the Information level:

```
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/1.1 GET https://localhost:5001/api/products
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'ProductsController.Get'
info: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[3]
      Route matched with {controller = "Products", action = "Get"}
info: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[102]
      Executing action result ObjectResult
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished HTTP/1.1 GET https://localhost:5001/api/products - 200 1234 application/json 45.6789ms
```

That is five log entries for a single request. In production, this is enormous noise.

## Serilog Request Logging -- One Entry Per Request

```csharp
// Suppress the noisy framework logs
builder.Host.UseSerilog((context, configuration) =>
{
    configuration
        .MinimumLevel.Override("Microsoft.AspNetCore", LogEventLevel.Warning)
        .WriteTo.Console();
});

var app = builder.Build();

// This single middleware replaces all of the above with one line:
app.UseSerilogRequestLogging();
```

Output:

```
[14:23:45 INF] HTTP GET /api/products responded 200 in 45.67 ms
```

One log entry per request with all the important information. You can enrich it with additional data:

```csharp
app.UseSerilogRequestLogging(options =>
{
    // Include the query string
    options.EnrichDiagnosticContext = (diagnosticContext, httpContext) =>
    {
        diagnosticContext.Set("QueryString", 
            httpContext.Request.QueryString.ToString());
        diagnosticContext.Set("ClientIp", 
            httpContext.Connection.RemoteIpAddress?.ToString());
        diagnosticContext.Set("UserAgent", 
            httpContext.Request.Headers.UserAgent.ToString());

        // Include the authenticated user if available
        if (httpContext.User.Identity?.IsAuthenticated == true)
        {
            diagnosticContext.Set("UserName", 
                httpContext.User.Identity.Name);
        }
    };

    // Customize the message template
    options.MessageTemplate = 
        "HTTP {RequestMethod} {RequestPath} responded {StatusCode} in {Elapsed:0.00} ms";

    // Only log errors for specific status codes
    options.GetLevel = (httpContext, elapsed, ex) =>
    {
        if (ex is not null || httpContext.Response.StatusCode >= 500)
            return LogEventLevel.Error;

        if (elapsed > 5000)  // Slow requests
            return LogEventLevel.Warning;

        return LogEventLevel.Information;
    };
});
```

> [!tip]
> Place `app.UseSerilogRequestLogging()` **after** `UseStaticFiles()` but **before** `UseRouting()`. This way, static file requests (CSS, JS, images) are not logged (they are handled before the logging middleware), but all API and MVC requests are captured.

> [!summary] Section Summary
> - Default ASP.NET Core logging produces multiple noisy entries per request
> - `app.UseSerilogRequestLogging()` replaces this with a single, structured log entry per request
> - Enrich the diagnostic context with query strings, client IP, user agent, and authenticated user
> - Use `GetLevel` to elevate slow requests to Warning and server errors to Error
> - Place request logging after `UseStaticFiles()` to skip logging static file requests
