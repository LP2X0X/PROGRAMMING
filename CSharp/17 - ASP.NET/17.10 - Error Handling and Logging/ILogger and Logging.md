---
tags: [csharp, asp-net-core, logging, ilogger, observability]
aliases: [ASP.NET Core Logging, ILogger, Structured Logging, Serilog Setup]
status: complete
date: 2026-06-18
---

# ILogger and Logging

Logging is the primary way you observe what your application is doing in production. When something goes wrong at 3 AM, you are not stepping through code with a debugger -- you are reading logs. The quality of your logging directly determines how quickly you can diagnose issues, understand user behavior, and verify that your application is functioning correctly.

ASP.NET Core provides a **built-in logging abstraction** (`ILogger<T>`) that decouples your application code from any specific logging framework. You write log statements against the abstraction, and the actual destination (console, file, Elasticsearch, Application Insights) is determined by configuration -- not by changing your code.

---

## Table of Contents

- [[#The Built-in Logging Abstraction]]
- [[#Log Levels -- When to Use Each]]
- [[#Structured Logging -- Why Placeholders Matter]]
- [[#Log Categories]]
- [[#Configuring Log Levels]]
- [[#Built-in Logging Providers]]
- [[#Third-Party Providers -- Serilog, NLog, log4net]]
- [[#Serilog Setup in Detail]]
- [[#Log Scopes -- Adding Context]]
- [[#Request Logging Middleware]]
- [[#High-Performance Logging]]
- [[#Correlation IDs]]
- [[#What NOT to Log]]
- [[#Real-World -- Complete Logging Setup]]
- [[#Related Topics]]
- [[#Further Reading]]
- [[#Comprehensive Summary]]

---

## The Built-in Logging Abstraction

ASP.NET Core includes a logging system in the `Microsoft.Extensions.Logging` namespace. The central interface is **`ILogger<T>`**, which you inject via [[DI Overview|dependency injection]] everywhere you need to log.

```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;

    public OrderService(ILogger<OrderService> logger)
    {
        _logger = logger;
    }

    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        _logger.LogInformation("Creating order for customer {CustomerId}", 
            request.CustomerId);

        var order = new Order { CustomerId = request.CustomerId };
        // ... business logic ...

        _logger.LogInformation("Order {OrderId} created successfully", 
            order.Id);

        return order;
    }
}
```

The key design decision behind `ILogger<T>` is the **abstraction layer**. Your code never references a specific logging library -- it only depends on `ILogger<T>`. This means you can switch from Console logging to Serilog to NLog without changing a single line of application code.

### The Logging Interfaces

| Interface | Purpose |
|---|---|
| `ILogger` | Base interface with `Log()`, `IsEnabled()`, `BeginScope()` |
| `ILogger<T>` | Generic version that sets the log **category** to the type name of `T` |
| `ILoggerFactory` | Creates `ILogger` instances; registered as a singleton in DI |
| `ILoggerProvider` | Represents a logging destination (Console, File, etc.) |

```csharp
// ILogger<T> is the most common -- inject it with the class type
public class ProductsController : Controller
{
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(ILogger<ProductsController> logger)
    {
        _logger = logger;
    }
}

// ILoggerFactory is useful when you need to create loggers dynamically
public class DynamicService
{
    private readonly ILoggerFactory _loggerFactory;

    public DynamicService(ILoggerFactory loggerFactory)
    {
        _loggerFactory = loggerFactory;
    }

    public void ProcessBatch(string batchName)
    {
        // Create a logger with a custom category name
        var logger = _loggerFactory.CreateLogger($"BatchProcessor.{batchName}");
        logger.LogInformation("Starting batch {BatchName}", batchName);
    }
}
```

> [!ad-note]
> `ILogger<T>` is registered in DI automatically when you call `WebApplication.CreateBuilder()`. You do not need to register it manually. The framework resolves it by creating an `ILogger` with the category set to the full name of `T`.

> [!summary] Section Summary
> - `ILogger<T>` is the primary logging interface, injected via DI throughout the application
> - It is an abstraction -- your code does not depend on any specific logging framework
> - `ILoggerFactory` creates loggers with custom category names for dynamic scenarios
> - The logging system is registered automatically by `WebApplication.CreateBuilder()`
> - This abstraction lets you switch logging providers (Console, Serilog, NLog) without changing application code

---

## Log Levels -- When to Use Each

ASP.NET Core defines six log levels, ordered from most verbose (Trace) to most severe (Critical). Choosing the right level is an art that directly affects the usefulness of your logs.

| Level | Numeric | When to Use | Example |
|---|---|---|---|
| **Trace** | 0 | Highly detailed diagnostic info; may contain sensitive data | `"Entering method GetProduct with id=42"` |
| **Debug** | 1 | Internal application flow useful during development | `"Cache miss for key 'product:42', fetching from DB"` |
| **Information** | 2 | General application flow -- significant business events | `"Order 1234 created for customer 567"` |
| **Warning** | 3 | Unexpected events that do not stop the application | `"Payment retry attempt 2 of 3 for order 1234"` |
| **Error** | 4 | An operation failed but the application continues | `"Failed to send confirmation email for order 1234"` |
| **Critical** | 5 | Application-wide failure requiring immediate attention | `"Database connection pool exhausted, no connections available"` |
| **None** | 6 | Disables logging entirely for a category (configuration only) | Used only in config, not in code |

### Detailed Guidance

**Trace** -- Use for method entry/exit, parameter values, and detailed diagnostic information. This level generates enormous volume and is typically only enabled when actively debugging a specific issue. Never log sensitive data even at Trace level if logs are centralized.

**Debug** -- Use for internal logic decisions: cache hits/misses, branch decisions, computed values. Useful during development and for diagnosing specific issues in production (by temporarily lowering the level for a category).

**Information** -- Use for significant business events: user logged in, order created, payment processed, report generated. These form the "story" of what your application is doing. In production, this is typically the default minimum level.

**Warning** -- Use when something unexpected happened but the application recovered or can continue. Retry attempts, deprecated API usage, configuration fallbacks, slow queries.

**Error** -- Use when an operation failed. The specific request could not be completed, but the application can still serve other requests. Include the exception object when available.

**Critical** -- Use for catastrophic failures: database unreachable, out of memory, unrecoverable state corruption. These should trigger immediate alerts. Most applications log Critical very rarely.

```csharp
public async Task<Order> ProcessOrderAsync(OrderRequest request)
{
    _logger.LogTrace("Entering ProcessOrderAsync with {@Request}", request);

    _logger.LogDebug("Checking inventory for {ProductCount} products", 
        request.Items.Count);

    if (await _inventory.ReserveAsync(request.Items))
    {
        _logger.LogInformation(
            "Order {OrderId} placed by customer {CustomerId} for {Total:C}",
            request.OrderId, request.CustomerId, request.Total);
    }
    else
    {
        _logger.LogWarning(
            "Insufficient inventory for order {OrderId}, " +
            "customer {CustomerId} will be notified",
            request.OrderId, request.CustomerId);
    }

    try
    {
        await _emailService.SendConfirmationAsync(request.Email);
    }
    catch (SmtpException ex)
    {
        // The order succeeded, but email failed -- not fatal
        _logger.LogError(ex,
            "Failed to send confirmation email for order {OrderId} " +
            "to {Email}",
            request.OrderId, request.Email);
    }

    _logger.LogTrace("Exiting ProcessOrderAsync");
    return order;
}
```

> [!warning] Common Misconception
> Many developers use `LogError` for any exception they catch, including expected business scenarios like "product not found" or "invalid input." These are **not errors** -- they are expected application behavior. Use Warning for expected-but-notable situations and reserve Error for genuine failures. Over-using Error causes alert fatigue and makes real errors harder to find.

> [!summary] Section Summary
> - Six log levels from Trace (most verbose) to Critical (most severe) control the verbosity of your logs
> - Information is the standard production level -- it captures the "story" of business events
> - Warning is for recoverable unexpected situations; Error is for operation failures
> - Critical is rare and should trigger immediate alerts
> - Choose levels carefully: over-logging at Error/Critical causes alert fatigue; under-logging at Information means missing context when debugging

---

## Structured Logging -- Why Placeholders Matter

This is one of the most important concepts in modern logging. **Structured logging** means log entries are not just text strings -- they are data records with named properties that can be searched, filtered, and aggregated.

### The Wrong Way -- String Interpolation

```csharp
// BAD: String interpolation
_logger.LogInformation($"Processing order {orderId} for customer {customerId}");
```

This produces a flat string: `"Processing order 42 for customer 567"`. The logging system sees it as a single pre-formatted message with no structure. You cannot search for "all logs where orderId = 42" because `orderId` is just part of a text string.

### The Right Way -- Message Templates

```csharp
// GOOD: Message template with named placeholders
_logger.LogInformation("Processing order {OrderId} for customer {CustomerId}",
    orderId, customerId);
```

This produces a log entry with:
- A **message template**: `"Processing order {OrderId} for customer {CustomerId}"`
- Named **properties**: `OrderId = 42`, `CustomerId = 567`
- A **rendered message**: `"Processing order 42 for customer 567"`

In a log aggregation system like Seq, Elasticsearch, or Application Insights, you can now query: `OrderId = 42` and find every log entry related to that order -- across all services, all log levels.

### How It Works Under the Hood

The logging framework parses the message template at call time. Each `{Placeholder}` becomes a named property in the log event. The values are passed as parameters (not interpolated into the string) and are stored as structured data alongside the rendered text.

```csharp
// The placeholder names become property names in the log event
_logger.LogInformation(
    "User {UserName} from {IpAddress} accessed {Endpoint} in {ElapsedMs}ms",
    userName,       // Property: UserName
    ipAddress,      // Property: IpAddress
    endpoint,       // Property: Endpoint
    elapsedMs       // Property: ElapsedMs
);

// In Seq or Elasticsearch, you can now query:
// UserName = "john.doe" AND ElapsedMs > 1000
// IpAddress = "10.0.1.42"
// Endpoint = "/api/orders" AND @Level = "Warning"
```

### Destructuring with @

For complex objects, prefix the placeholder with `@` to serialize the object's properties instead of calling `.ToString()`:

```csharp
var order = new { Id = 42, CustomerId = 567, Total = 99.99 };

// Without @: logs order.ToString() which is "{ Id = 42, CustomerId = 567, Total = 99.99 }"
_logger.LogInformation("Processing order {Order}", order);

// With @: destructures the object into individual properties
_logger.LogInformation("Processing order {@Order}", order);
// Creates properties: Order.Id = 42, Order.CustomerId = 567, Order.Total = 99.99
```

> [!danger]
> ==Never use string interpolation (`$"..."`) with logging methods.== Besides losing structure, interpolated strings are always formatted -- even if the log level is disabled. This wastes CPU and allocations. With message templates, the formatting only happens if the log level is enabled.

> [!tip]
> Adopt consistent naming conventions for your log properties across the entire application. Always use `{OrderId}`, not sometimes `{orderId}` and sometimes `{order_id}`. Consistent names make cross-service log correlation possible.

> [!summary] Section Summary
> - Structured logging uses **message templates** with `{NamedPlaceholders}` instead of string interpolation
> - Named placeholders become searchable, filterable properties in log aggregation systems
> - Use `@` prefix to destructure complex objects into their individual properties
> - Never use `$"..."` string interpolation -- it destroys structure and wastes resources when the level is disabled
> - Consistent property naming across services enables powerful cross-service log correlation

---

## Log Categories

Every `ILogger` instance has a **category** -- a string that identifies the source of the log entry. When you inject `ILogger<ProductsController>`, the category is the full type name: `"MyApp.Controllers.ProductsController"`.

Categories serve two purposes:
1. **Identification** -- you can see which class generated each log entry
2. **Filtering** -- you can configure different log levels per category

```csharp
// Category: "MyApp.Controllers.ProductsController"
public class ProductsController : Controller
{
    private readonly ILogger<ProductsController> _logger;
    // ...
}

// Category: "MyApp.Services.OrderService"
public class OrderService
{
    private readonly ILogger<OrderService> _logger;
    // ...
}

// Custom category name (rare, but useful for cross-cutting concerns)
public class CacheService
{
    private readonly ILogger _logger;

    public CacheService(ILoggerFactory factory)
    {
        // Category: "Caching"
        _logger = factory.CreateLogger("Caching");
    }
}
```

> [!ad-note]
> The category string uses the namespace hierarchy, which is why per-category filtering works with prefixes. Setting `"MyApp.Services": "Warning"` in configuration affects all loggers in the `MyApp.Services` namespace.

> [!summary] Section Summary
> - Log categories identify the source of each log entry and enable per-source filtering
> - `ILogger<T>` automatically uses the full type name of `T` as the category
> - Use `ILoggerFactory.CreateLogger("name")` for custom category names
> - Categories follow the namespace hierarchy, enabling prefix-based filtering

---

## Configuring Log Levels

Log level filtering is configured in `appsettings.json` under the `"Logging"` section. You can set different minimum levels per category prefix.

### Default Configuration

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore": "Warning",
      "System": "Warning"
    }
  }
}
```

This configuration means:
- **All application code**: Information and above (Info, Warning, Error, Critical)
- **ASP.NET Core framework**: Warning and above (suppress the noisy Info logs from routing, hosting, etc.)
- **Entity Framework Core**: Warning and above (suppress the SQL query logs at Info level)
- **System namespace**: Warning and above

### Environment-Specific Configuration

Use `appsettings.Development.json` to configure more verbose logging during development:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Information",
      "Microsoft.EntityFrameworkCore.Database.Command": "Information",
      "MyApp": "Trace"
    }
  }
}
```

In development, this shows:
- Debug-level logs by default
- ASP.NET Core Information logs (request routing, middleware execution)
- EF Core SQL commands (the actual SQL queries being executed)
- Trace-level logs for your application code

### Per-Provider Configuration

You can also configure log levels per logging provider. This lets you send verbose logs to one destination and only errors to another:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    },
    "Console": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Debug": {
      "LogLevel": {
        "Default": "Debug"
      }
    }
  }
}
```

### Filtering Precedence

The filtering rules follow this precedence (first match wins):
1. Provider-specific category match (e.g., `"Console" > "MyApp.Services"`)
2. Provider-specific default (e.g., `"Console" > "Default"`)
3. Global category match (e.g., `"LogLevel" > "MyApp.Services"`)
4. Global default (e.g., `"LogLevel" > "Default"`)

> [!tip]
> When debugging a specific issue in production, you can temporarily lower the log level for just the relevant namespace without drowning in framework noise. For example, set `"MyApp.Services.PaymentService": "Debug"` while keeping everything else at Warning. If you use `appsettings.json` with `reloadOnChange: true` (the default), changes take effect without restarting the application.

> [!warning] Common Misconception
> Setting `"Default": "Trace"` in production is not "just more logs." Trace and Debug levels can generate thousands of log entries per second, overwhelming your log storage, degrading application performance, and potentially logging sensitive data that Trace-level code was never expected to hide. Always be deliberate about production log levels.

> [!summary] Section Summary
> - Configure log levels in `appsettings.json` under `"Logging"` > `"LogLevel"`
> - Use category prefixes for granular control: `"Microsoft.AspNetCore": "Warning"` suppresses noisy framework logs
> - Override per environment using `appsettings.Development.json` for verbose development logging
> - Per-provider configuration lets you send different verbosity levels to different destinations
> - Changes to `appsettings.json` take effect at runtime without restart (with `reloadOnChange: true`)

---

## Built-in Logging Providers

ASP.NET Core ships with several built-in logging providers:

| Provider | NuGet Package | Description |
|---|---|---|
| **Console** | Included | Writes to stdout -- visible in terminal, Docker logs, `kubectl logs` |
| **Debug** | Included | Writes to `System.Diagnostics.Debug` -- visible in VS Output window |
| **EventSource** | Included | Writes to ETW (Windows) / EventPipe (cross-platform) for diagnostic tools |
| **EventLog** | `Microsoft.Extensions.Logging.EventLog` | Writes to Windows Event Log |

```csharp
var builder = WebApplication.CreateBuilder(args);

// These are added automatically by CreateBuilder():
// - Console
// - Debug
// - EventSource
// - EventLog (Windows only)

// You can clear and re-add selectively:
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();
```

> [!ad-note]
> The Console provider in production (especially Docker/Kubernetes) is important because container orchestrators capture stdout. This is how `kubectl logs`, Docker Desktop, and cloud logging services (AWS CloudWatch, Azure Container Instances logs) ingest your application logs automatically.

> [!summary] Section Summary
> - Four built-in providers: Console, Debug, EventSource, EventLog (Windows)
> - `WebApplication.CreateBuilder()` registers Console, Debug, and EventSource automatically
> - Console provider is essential for containerized deployments where stdout is captured by the orchestrator
> - Use `ClearProviders()` and `Add*()` to customize which providers are active

---

## Third-Party Providers -- Serilog, NLog, log4net

The built-in providers handle basic scenarios, but production applications typically use a third-party logging library for features like file logging, log rotation, structured JSON output, and integration with log aggregation services.

| Library | Strengths | Ecosystem |
|---|---|---|
| **Serilog** | Best structured logging support, rich sink ecosystem, message templates | 200+ sinks (Seq, Elasticsearch, Application Insights, etc.) |
| **NLog** | Mature, flexible targeting rules, strong XML configuration | Rich target ecosystem, familiar to classic .NET developers |
| **log4net** | Port of Java's log4j, very mature | Legacy choice -- less active development |

**Serilog** is by far the most popular choice for new ASP.NET Core applications due to its first-class support for structured logging and its enormous ecosystem of "sinks" (output destinations).

> [!tip]
> For new projects, choose **Serilog**. It was designed from the ground up for structured logging, which is exactly what modern observability tools (Seq, Elasticsearch/ELK, Datadog, Splunk) expect. NLog is solid but Serilog's structured data model is more native to the message template approach already built into `ILogger<T>`.

> [!summary] Section Summary
> - Third-party providers add file logging, JSON output, log rotation, and integration with log aggregation services
> - Serilog is the most popular choice for ASP.NET Core due to native structured logging support
> - NLog is a mature alternative with strong XML configuration support
> - log4net is primarily used in legacy applications migrating from classic .NET

---

## Serilog Setup in Detail

Serilog integrates with ASP.NET Core by replacing the built-in logging pipeline. Here is a complete production-ready setup.

### NuGet Packages

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.Sinks.Seq
dotnet add package Serilog.Enrichers.Environment
dotnet add package Serilog.Enrichers.Thread
dotnet add package Serilog.Settings.Configuration
```

### Program.cs Configuration

```csharp
using Serilog;

// Configure Serilog early to capture startup errors
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.Console()
    .CreateBootstrapLogger();

try
{
    Log.Information("Starting up the application");

    var builder = WebApplication.CreateBuilder(args);

    // Replace the built-in logging with Serilog
    builder.Host.UseSerilog((context, services, configuration) =>
    {
        configuration
            .ReadFrom.Configuration(context.Configuration)
            .ReadFrom.Services(services)
            .Enrich.FromLogContext()
            .Enrich.WithMachineName()
            .Enrich.WithThreadId()
            .Enrich.WithProperty("Application", "MyApp");
    });

    // ... register services ...

    var app = builder.Build();

    // Add Serilog request logging (replaces the built-in HTTP logging)
    app.UseSerilogRequestLogging(options =>
    {
        // Customize what is logged per request
        options.EnrichDiagnosticContext = (diagnosticContext, httpContext) =>
        {
            diagnosticContext.Set("RequestHost", httpContext.Request.Host.Value);
            diagnosticContext.Set("UserAgent", 
                httpContext.Request.Headers.UserAgent.ToString());
        };
    });

    // ... configure pipeline ...

    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}
```

### appsettings.json Serilog Configuration

```json
{
  "Serilog": {
    "Using": ["Serilog.Sinks.Console", "Serilog.Sinks.File", "Serilog.Sinks.Seq"],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft.AspNetCore": "Warning",
        "Microsoft.EntityFrameworkCore": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "theme": "Serilog.Sinks.SystemConsole.Themes.AnsiConsoleTheme::Code, Serilog.Sinks.Console",
          "outputTemplate": "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj} {Properties:j}{NewLine}{Exception}"
        }
      },
      {
        "Name": "File",
        "Args": {
          "path": "logs/app-.log",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 30,
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level:u3}] [{SourceContext}] {Message:lj}{NewLine}{Exception}",
          "fileSizeLimitBytes": 104857600
        }
      },
      {
        "Name": "Seq",
        "Args": {
          "serverUrl": "http://localhost:5341"
        }
      }
    ],
    "Enrich": ["FromLogContext", "WithMachineName", "WithThreadId"]
  }
}
```

### Serilog Enrichers

Enrichers automatically add properties to every log event:

| Enricher | Property Added | Package |
|---|---|---|
| `FromLogContext` | Any property pushed to `LogContext` | Included in Serilog |
| `WithMachineName` | `MachineName` | Serilog.Enrichers.Environment |
| `WithEnvironmentName` | `EnvironmentName` | Serilog.Enrichers.Environment |
| `WithThreadId` | `ThreadId` | Serilog.Enrichers.Thread |
| `WithClientIp` | `ClientIp` | Serilog.Enrichers.ClientInfo |
| `WithCorrelationId` | `CorrelationId` | Serilog.Enrichers.CorrelationId |

### Serilog Sinks

Common sinks (output destinations):

| Sink | Description |
|---|---|
| **Console** | Colorized console output; essential for containers |
| **File** | Rolling file logs with size limits and retention |
| **Seq** | Purpose-built structured log server with a query language |
| **Elasticsearch** | Logs to Elasticsearch for Kibana dashboards |
| **ApplicationInsights** | Azure Application Insights integration |
| **Datadog** | Datadog logging integration |
| **Splunk** | Splunk HTTP Event Collector |

> [!ad-note]
> The **bootstrap logger** pattern (`CreateBootstrapLogger()` followed by `UseSerilog()`) is important. The bootstrap logger captures any errors that occur during startup configuration (before the full pipeline is built). Without it, a configuration error during startup could crash the application silently with no log output.

> [!summary] Section Summary
> - Serilog replaces the built-in logging via `builder.Host.UseSerilog()`
> - Use the bootstrap logger pattern to capture startup errors before the full pipeline is configured
> - Configure sinks (Console, File, Seq, Elasticsearch) and enrichers (MachineName, ThreadId) in `appsettings.json`
> - The `Serilog.Settings.Configuration` package enables full JSON-based configuration
> - Rolling file logs with retention limits prevent disk space exhaustion in production

---

## Log Scopes -- Adding Context

**Log scopes** add contextual properties to all log entries within a block of code. This is invaluable for tracing a request through multiple service calls.

```csharp
public async Task<Order> ProcessOrderAsync(int orderId, int customerId)
{
    // All log entries within this using block will include
    // OrderId and CustomerId as properties
    using (_logger.BeginScope(
        new Dictionary<string, object>
        {
            ["OrderId"] = orderId,
            ["CustomerId"] = customerId
        }))
    {
        _logger.LogInformation("Starting order processing");
        // Log entry includes: OrderId=42, CustomerId=567

        await ValidateInventory();
        // Any logs inside ValidateInventory also include OrderId and CustomerId

        await ProcessPayment();
        // Same here -- the scope follows the async execution context

        _logger.LogInformation("Order processing completed");
    }
    // Scope ends here -- OrderId and CustomerId are no longer added
}
```

You can also use simpler string-based scopes:

```csharp
using (_logger.BeginScope("Processing order {OrderId}", orderId))
{
    _logger.LogInformation("Validating inventory");
    _logger.LogInformation("Processing payment");
}
```

### Enabling Scopes in Configuration

For the Console provider, scopes are disabled by default. Enable them:

```json
{
  "Logging": {
    "Console": {
      "IncludeScopes": true
    }
  }
}
```

> [!ad-note]
> Serilog automatically includes scope properties when you use the `FromLogContext` enricher (which you should always include). No additional configuration is needed.

> [!summary] Section Summary
> - `BeginScope` adds contextual properties to all log entries within the `using` block
> - Scopes follow async execution -- they work correctly across `await` boundaries
> - Use dictionary scopes for named properties or string scopes for simple context
> - Console provider requires `"IncludeScopes": true`; Serilog handles scopes automatically via `FromLogContext`

---

## Request Logging Middleware

Every HTTP request should generate at least one log entry with the method, path, status code, and response time. Instead of building this yourself, use **Serilog's request logging middleware**.

### The Problem with Default Framework Logging

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

### Serilog Request Logging -- One Entry Per Request

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

---

## High-Performance Logging

For hot paths (code that executes thousands of times per second), the overhead of parsing message templates and boxing value-type arguments on every call can become measurable. .NET provides **`LoggerMessage.Define`** for zero-allocation, pre-compiled log delegates.

### LoggerMessage.Define (Pre-.NET 8)

```csharp
public partial class OrderService
{
    // Pre-compiled log delegates -- message template is parsed once
    private static readonly Action<ILogger, int, string, Exception?> _orderCreated =
        LoggerMessage.Define<int, string>(
            LogLevel.Information,
            new EventId(1001, nameof(OrderCreated)),
            "Order {OrderId} created for customer {CustomerName}");

    private static readonly Action<ILogger, int, Exception?> _orderFailed =
        LoggerMessage.Define<int>(
            LogLevel.Error,
            new EventId(1002, nameof(OrderFailed)),
            "Failed to create order for customer {CustomerId}");

    private void OrderCreated(int orderId, string customerName)
        => _orderCreated(_logger, orderId, customerName, null);

    private void OrderFailed(int customerId, Exception ex)
        => _orderFailed(_logger, customerId, ex);
}
```

### Source Generators (C# 12 / .NET 8+)

.NET 8 introduced the **`[LoggerMessage]` attribute** with source generators, which is much cleaner:

```csharp
public partial class OrderService
{
    private readonly ILogger<OrderService> _logger;

    [LoggerMessage(
        EventId = 1001,
        Level = LogLevel.Information,
        Message = "Order {OrderId} created for customer {CustomerName}")]
    partial void LogOrderCreated(int orderId, string customerName);

    [LoggerMessage(
        EventId = 1002,
        Level = LogLevel.Error,
        Message = "Failed to create order for customer {CustomerId}")]
    partial void LogOrderFailed(int customerId, Exception ex);

    public async Task CreateOrderAsync(int customerId, string customerName)
    {
        // Use like any other method -- zero allocation, pre-compiled
        var orderId = await _repository.CreateAsync(customerId);
        LogOrderCreated(orderId, customerName);
    }
}
```

> [!ad-note]
> The class must be declared `partial` for source generators to work. The generated code handles the `IsEnabled()` check, avoids boxing value types, and pre-parses the message template -- all at compile time.

### When to Use High-Performance Logging

| Scenario | Regular Logging | High-Performance Logging |
|---|---|---|
| Controller actions | Yes | Overkill |
| Service-layer business logic | Yes | Usually unnecessary |
| Inner loops processing thousands of items | No | Yes |
| Middleware on every request | Yes for most | Yes if the app handles 10K+ RPS |
| Library code shared across many applications | No | Yes |

> [!summary] Section Summary
> - `LoggerMessage.Define` and `[LoggerMessage]` source generators provide zero-allocation, pre-compiled logging
> - The source generator approach (`.NET 8+`) is cleaner and handles the `IsEnabled()` check automatically
> - Use high-performance logging in hot paths (inner loops, high-RPS middleware, library code)
> - For normal application code (controllers, services), standard `ILogger` methods are fine

---

## Correlation IDs

A **correlation ID** (also called request ID or trace ID) is a unique identifier that follows a request through all the services and log entries it generates. This is essential for debugging in distributed systems where a single user action might hit multiple microservices.

### ASP.NET Core Built-in TraceIdentifier

ASP.NET Core automatically generates a `TraceIdentifier` for each request:

```csharp
// Every HttpContext has a TraceIdentifier
var traceId = context.TraceIdentifier;
// Example: "0HMVK2K9M8RHS:00000001"
```

### Custom Correlation ID Middleware

For cross-service tracing, use a custom correlation ID that can be passed between services:

```csharp
public class CorrelationIdMiddleware
{
    private const string CorrelationIdHeader = "X-Correlation-Id";
    private readonly RequestDelegate _next;

    public CorrelationIdMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Check if the caller (e.g., an upstream service) sent a correlation ID
        if (!context.Request.Headers.TryGetValue(
            CorrelationIdHeader, out var correlationId)
            || string.IsNullOrWhiteSpace(correlationId))
        {
            // Generate a new one if not provided
            correlationId = Guid.NewGuid().ToString("N");
        }

        // Store in HttpContext.Items for use in the current request
        context.Items["CorrelationId"] = correlationId.ToString();

        // Add to response headers so the client can use it for support requests
        context.Response.OnStarting(() =>
        {
            context.Response.Headers[CorrelationIdHeader] = correlationId;
            return Task.CompletedTask;
        });

        // Push to Serilog's LogContext so every log entry includes it
        using (LogContext.PushProperty("CorrelationId", correlationId.ToString()))
        {
            await _next(context);
        }
    }
}
```

### Propagating Correlation IDs to Downstream Services

When calling other services, forward the correlation ID:

```csharp
public class ExternalApiClient
{
    private readonly HttpClient _httpClient;
    private readonly IHttpContextAccessor _httpContextAccessor;

    public ExternalApiClient(
        HttpClient httpClient,
        IHttpContextAccessor httpContextAccessor)
    {
        _httpClient = httpClient;
        _httpContextAccessor = httpContextAccessor;
    }

    public async Task<Order> GetOrderAsync(int orderId)
    {
        var correlationId = _httpContextAccessor.HttpContext?
            .Items["CorrelationId"]?.ToString();

        var request = new HttpRequestMessage(HttpMethod.Get,
            $"/api/orders/{orderId}");

        if (correlationId is not null)
        {
            request.Headers.Add("X-Correlation-Id", correlationId);
        }

        var response = await _httpClient.SendAsync(request);
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<Order>();
    }
}
```

> [!tip]
> For production distributed tracing, consider using **OpenTelemetry** instead of custom correlation ID middleware. OpenTelemetry is an industry standard that provides distributed tracing, metrics, and logging -- with automatic context propagation using W3C Trace Context headers. Serilog integrates with OpenTelemetry via `Serilog.Enrichers.Span`.

> [!summary] Section Summary
> - Correlation IDs are unique identifiers that follow a request through all services and log entries
> - ASP.NET Core provides `HttpContext.TraceIdentifier` automatically, but custom IDs offer more control
> - Use middleware to extract or generate the ID, store it in `HttpContext.Items`, push it to `LogContext`
> - Forward the ID in outbound HTTP requests to downstream services
> - For production distributed systems, consider OpenTelemetry for industry-standard tracing

---

## What NOT to Log

Logging sensitive data is a security and compliance violation. Your logs are stored in log files, log aggregation services, and potentially backed up -- all of which expand the attack surface.

### Never Log These

| Data Type | Why It Is Dangerous |
|---|---|
| **Passwords / secrets** | Even hashed passwords should not be in logs; plain text is catastrophic |
| **Authentication tokens** | JWT tokens, API keys, session tokens -- anyone with log access can impersonate users |
| **Credit card numbers** | PCI-DSS compliance violation; can result in fines and loss of payment processing |
| **Social Security Numbers** | PII laws (GDPR, CCPA) make this a compliance violation |
| **Medical records** | HIPAA violation (if applicable) |
| **Full request/response bodies** | May contain any of the above in POST requests or API responses |
| **Connection strings** | Contain database credentials |
| **Encryption keys** | Obvious compromise |

### Defensive Patterns

```csharp
// BAD: Logging the full request body
_logger.LogInformation("Received: {Body}", await ReadBodyAsync(context));

// BAD: Logging authentication headers
_logger.LogDebug("Auth header: {Auth}", context.Request.Headers.Authorization);

// BAD: Logging user input that might contain sensitive data
_logger.LogInformation("User submitted: {FormData}", request.ToString());

// GOOD: Log only safe, identifying information
_logger.LogInformation(
    "Login attempt for user {Username} from {IpAddress}",
    request.Username,  // Username is safe to log
    context.Connection.RemoteIpAddress);

// GOOD: Mask sensitive data if you must log it for debugging
_logger.LogDebug(
    "Processing card ending in {CardLast4} for {Amount:C}",
    cardNumber[^4..],  // Only last 4 digits
    amount);
```

> [!danger]
> Do not rely on log level filtering to protect sensitive data. "We only log card numbers at Trace level, and Trace is disabled in production" is not a security measure. Log levels can be changed at runtime, and a developer troubleshooting an issue might enable Trace without realizing what gets exposed. ==Never write sensitive data to log statements at any level.==

### Serilog Destructure Policies

Serilog can automatically mask sensitive properties during destructuring:

```csharp
builder.Host.UseSerilog((context, configuration) =>
{
    configuration
        .Destructure.ByTransforming<LoginRequest>(
            r => new { r.Username, Password = "***" })
        .Destructure.ByTransforming<PaymentRequest>(
            r => new { r.OrderId, r.Amount, CardNumber = "***" });
});
```

> [!summary] Section Summary
> - Never log passwords, tokens, credit cards, SSNs, connection strings, or encryption keys at any level
> - Do not rely on log level filtering as a security measure -- levels can change at runtime
> - Mask sensitive fields if partial information is needed (e.g., last 4 digits of a card)
> - Use Serilog's destructure policies to automatically redact sensitive properties from objects
> - Logging full request/response bodies is dangerous because they may contain any sensitive data

---

## Real-World -- Complete Logging Setup

Here is a production-ready logging setup that combines everything covered in this note: Serilog, structured logging, request logging, correlation IDs, and environment-specific configuration.

### Program.cs

```csharp
using Serilog;
using Serilog.Events;

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .CreateBootstrapLogger();

try
{
    Log.Information("Starting application");

    var builder = WebApplication.CreateBuilder(args);

    builder.Host.UseSerilog((context, services, configuration) =>
        configuration.ReadFrom.Configuration(context.Configuration)
            .ReadFrom.Services(services));

    builder.Services.AddHttpContextAccessor();
    builder.Services.AddControllersWithViews();
    builder.Services.AddProblemDetails();
    builder.Services.AddExceptionHandler<GlobalExceptionHandler>();

    var app = builder.Build();

    if (!app.Environment.IsDevelopment())
    {
        app.UseExceptionHandler();
        app.UseHsts();
    }

    app.UseHttpsRedirection();

    // Correlation ID middleware -- early in pipeline
    app.UseMiddleware<CorrelationIdMiddleware>();

    app.UseStaticFiles();

    // Serilog request logging -- after static files, before routing
    app.UseSerilogRequestLogging(options =>
    {
        options.EnrichDiagnosticContext = (diagnosticContext, httpContext) =>
        {
            diagnosticContext.Set("ClientIp",
                httpContext.Connection.RemoteIpAddress?.ToString());
            diagnosticContext.Set("UserAgent",
                httpContext.Request.Headers.UserAgent.ToString());

            if (httpContext.User.Identity?.IsAuthenticated == true)
                diagnosticContext.Set("UserName",
                    httpContext.User.Identity.Name);
        };

        options.GetLevel = (httpContext, elapsed, ex) =>
        {
            if (ex is not null) return LogEventLevel.Error;
            if (httpContext.Response.StatusCode >= 500) return LogEventLevel.Error;
            if (elapsed > 5000) return LogEventLevel.Warning;
            if (httpContext.Response.StatusCode >= 400) return LogEventLevel.Warning;
            return LogEventLevel.Information;
        };
    });

    app.UseRouting();
    app.UseAuthentication();
    app.UseAuthorization();
    app.MapControllers();

    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Application terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}
```

### appsettings.json

```json
{
  "Serilog": {
    "Using": ["Serilog.Sinks.Console", "Serilog.Sinks.File"],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft.AspNetCore": "Warning",
        "Microsoft.EntityFrameworkCore": "Warning",
        "System.Net.Http.HttpClient": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "outputTemplate": "[{Timestamp:HH:mm:ss} {Level:u3}] [{CorrelationId}] {Message:lj}{NewLine}{Exception}"
        }
      },
      {
        "Name": "File",
        "Args": {
          "path": "logs/app-.log",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 30,
          "fileSizeLimitBytes": 104857600
        }
      }
    ],
    "Enrich": ["FromLogContext", "WithMachineName", "WithThreadId"]
  }
}
```

### appsettings.Development.json

```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Debug",
      "Override": {
        "Microsoft.AspNetCore": "Information",
        "Microsoft.EntityFrameworkCore.Database.Command": "Information"
      }
    }
  }
}
```

> [!summary] Section Summary
> - The complete setup combines Serilog, correlation IDs, request logging, and [[Exception Handling|global exception handling]]
> - Bootstrap logger captures startup errors; `Log.CloseAndFlush()` ensures all logs are written before shutdown
> - Request logging is enriched with client IP, user agent, and authenticated user name
> - Slow requests (>5s) are elevated to Warning; server errors to Error
> - Environment-specific configuration provides verbose Debug logging in development and focused Information logging in production

---

## Related Topics

- [[Exception Handling]] -- global exception handling that catches and logs unhandled exceptions
- [[Problem Details]] -- standardized error response format that includes trace IDs for log correlation
- [[Middleware Overview]] -- how logging middleware fits in the request pipeline
- [[DI Overview]] -- how `ILogger<T>` is resolved through dependency injection
- [[Configuration Overview]] -- how `appsettings.json` drives logging configuration

---

## Further Reading

- [[Health Checks]] -- monitoring application health and logging health check results
- [[OpenTelemetry]] -- industry-standard distributed tracing, metrics, and logging
- [[Application Performance Monitoring]] -- APM tools that build on top of structured logging
- [[Minimal APIs]] -- logging patterns in minimal API endpoints

---

## Comprehensive Summary

> [!tip] Complete Summary
> **ASP.NET Core logging** is built on the `ILogger<T>` abstraction from `Microsoft.Extensions.Logging`. This interface is injected via DI and decouples your application code from any specific logging framework, allowing you to switch providers without code changes.
>
> **Six log levels** (Trace through Critical) control verbosity. Information is the standard production level for business events. Warning is for recoverable surprises. Error is for operation failures. Critical is rare and should trigger alerts. Choosing the wrong level causes either alert fatigue (overuse of Error) or missing context (underuse of Information).
>
> **Structured logging** is the most important concept: use message templates with `{NamedPlaceholders}` instead of string interpolation. Placeholders become searchable properties in log aggregation systems. Never use `$"..."` -- it destroys structure and wastes resources.
>
> **Log categories** (`ILogger<T>` uses the type name) enable per-namespace filtering in `appsettings.json`. Suppress noisy framework logs with `"Microsoft.AspNetCore": "Warning"` while keeping your application logs at Information or Debug.
>
> **Serilog** is the de facto standard third-party provider, offering 200+ sinks (Console, File, Seq, Elasticsearch), enrichers (MachineName, ThreadId, CorrelationId), and full `appsettings.json` configuration. The bootstrap logger pattern captures startup errors. `UseSerilogRequestLogging()` replaces multiple noisy framework entries with a single structured log entry per request.
>
> **Log scopes** (`BeginScope`) add contextual properties to all log entries within a block. **Correlation IDs** propagate a unique request identifier across services for distributed tracing. **High-performance logging** via `[LoggerMessage]` source generators provides zero-allocation logging for hot paths.
>
> **Security is non-negotiable**: never log passwords, tokens, credit cards, or PII at any level. Do not rely on log level filtering as a security measure. Use Serilog destructure policies to automatically redact sensitive fields.