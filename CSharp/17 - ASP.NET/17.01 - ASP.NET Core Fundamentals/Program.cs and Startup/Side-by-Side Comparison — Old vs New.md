---
tags: [csharp, asp-net-core, startup, program]
---


| Aspect | Pre-.NET 6 (Startup.cs) | .NET 6+ (Minimal Hosting) |
|---|---|---|
| **Files** | `Program.cs` + `Startup.cs` | Single `Program.cs` |
| **Entry point** | `Main` method with `CreateHostBuilder` | Top-level statements |
| **Service registration** | `Startup.ConfigureServices(IServiceCollection)` | `builder.Services.Add*()` |
| **Pipeline config** | `Startup.Configure(IApplicationBuilder, IWebHostEnvironment)` | `app.Use*()` directly |
| **Host builder** | `Host.CreateDefaultBuilder(args)` | `WebApplication.CreateBuilder(args)` |
| **Endpoint mapping** | `app.UseEndpoints(e => e.MapControllers())` | `app.MapControllers()` directly |
| **Environment check** | Inject `IWebHostEnvironment env` parameter | `app.Environment.IsDevelopment()` |
| **Configuration access** | Inject `IConfiguration` in constructor | `builder.Configuration` directly |

> [!ad-note] Can You Still Use Startup.cs in .NET 6+?
> Yes. The minimal hosting model is the **default**, but you can still use the `Startup` class pattern if you prefer. However, the .NET templates no longer scaffold it, and the community has largely adopted minimal hosting.

> [!summary] Section Summary
> - The two-file model and single-file model are functionally equivalent.
> - The minimal model reduces boilerplate without sacrificing capability.
> - Service registration and pipeline configuration follow the same patterns in both.
> - The `Startup.cs` class pattern is still supported but no longer the default.
