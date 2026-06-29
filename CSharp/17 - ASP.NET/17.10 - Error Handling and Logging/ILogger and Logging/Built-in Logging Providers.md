---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---


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
