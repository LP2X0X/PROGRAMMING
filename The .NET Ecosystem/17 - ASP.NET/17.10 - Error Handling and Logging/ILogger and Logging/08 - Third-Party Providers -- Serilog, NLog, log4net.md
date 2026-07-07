---
tags:
  - csharp
  - asp-net-core
  - logging
  - ilogger
  - observability
---


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
