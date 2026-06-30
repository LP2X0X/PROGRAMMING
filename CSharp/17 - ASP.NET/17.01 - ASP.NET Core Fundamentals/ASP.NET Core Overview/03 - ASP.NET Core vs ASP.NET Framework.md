---
tags: [csharp, asp-net-core, fundamentals, web]
---


> [!warning] Migration Decision
> If you are maintaining an existing ASP.NET Framework app, you do not have to migrate. But all **new** projects should use ASP.NET Core. ASP.NET Framework is in maintenance mode -- it receives security patches only.

| Feature | ASP.NET Framework | ASP.NET Core |
|---|---|---|
| **Platform** | Windows only | Windows, Linux, macOS |
| **Web Server** | IIS only | Kestrel (cross-platform) + IIS/Nginx/Apache as reverse proxy |
| **Dependency Injection** | Not built-in (third-party required) | Built-in DI container |
| **Configuration** | `web.config` / `ConfigurationManager` | JSON, env vars, command line, user secrets, Azure Key Vault |
| **Performance** | Moderate | Top-tier (TechEmpower benchmarks) |
| **Pipeline** | `HttpModule` / `HttpHandler` (monolithic) | Middleware pipeline (composable) |
| **Hosting** | IIS process (`w3wp.exe`) | Self-hosted, IIS, Docker, systemd |
| **Side-by-Side** | Machine-wide .NET Framework | App-local .NET runtime |
| **Open Source** | Partially | Fully open source (MIT) |
| **Development Status** | Maintenance mode (security patches only) | Active development |
| **Deployment** | Windows Server + IIS | Anywhere: cloud, containers, edge |
| **Minimal APIs** | Not available | Supported (.NET 6+) |
| **Native AOT** | Not available | Supported (.NET 8+) |

> [!ad-note] What Cannot Be Migrated Easily
> Some ASP.NET Framework technologies have **no direct equivalent** in ASP.NET Core:
> - **Web Forms** -- no equivalent; use Blazor or Razor Pages instead
> - **WCF Server** -- use gRPC or REST APIs instead (CoreWCF exists as a community port)
> - **Windows-specific APIs** -- `System.Drawing`, `System.DirectoryServices`, etc. require Windows Compatibility Pack or alternatives

> [!summary] Section Summary
> - ASP.NET Core wins on platform support, performance, DI, and modern tooling
> - ASP.NET Framework is Windows/IIS-only and in maintenance mode
> - New projects should always target ASP.NET Core
> - Some legacy technologies (Web Forms, WCF) have no direct ASP.NET Core equivalent
> - Migration is possible but requires rewriting, not just upgrading
