---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---


Serilog integrates with ASP.NET Core by replacing the built-in logging pipeline. Here is a complete production-ready setup.

## NuGet Packages

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.Sinks.Seq
dotnet add package Serilog.Enrichers.Environment
dotnet add package Serilog.Enrichers.Thread
dotnet add package Serilog.Settings.Configuration
```

## Program.cs Configuration

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

## appsettings.json Serilog Configuration

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

## Serilog Enrichers

Enrichers automatically add properties to every log event:

| Enricher | Property Added | Package |
|---|---|---|
| `FromLogContext` | Any property pushed to `LogContext` | Included in Serilog |
| `WithMachineName` | `MachineName` | Serilog.Enrichers.Environment |
| `WithEnvironmentName` | `EnvironmentName` | Serilog.Enrichers.Environment |
| `WithThreadId` | `ThreadId` | Serilog.Enrichers.Thread |
| `WithClientIp` | `ClientIp` | Serilog.Enrichers.ClientInfo |
| `WithCorrelationId` | `CorrelationId` | Serilog.Enrichers.CorrelationId |

## Serilog Sinks

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
