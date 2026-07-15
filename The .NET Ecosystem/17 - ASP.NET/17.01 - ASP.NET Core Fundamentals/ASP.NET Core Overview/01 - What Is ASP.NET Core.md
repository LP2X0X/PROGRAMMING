---
tags: [csharp, asp-net-core, fundamentals, web]
---


ASP.NET Core is a **cross-platform, high-performance, open-source framework** for building modern, cloud-enabled web applications and services. You can use ASP.NET Core to build server-rendered web applications (Razor), backend server applications, HTTP APIs (Minimal API) that can be consumed by mobile applications, and much more. It runs on .NET (formerly .NET Core) and is designed from the ground up to be modular, testable, and lightweight.

Unlike its predecessor (ASP.NET on .NET Framework), ASP.NET Core is not tied to Windows or IIS. You can run it on Windows, Linux, and macOS, deploy it to Docker containers, and host it behind any reverse proxy -- not just IIS.

```csharp
// The simplest possible ASP.NET Core application
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello from ASP.NET Core!");

app.Run();
```

> [!tip] Minimal APIs
> Starting with .NET 6, ASP.NET Core supports **Minimal APIs** -- a streamlined way to build HTTP endpoints with minimal ceremony. The example above is a complete, runnable web application in just four lines.

### Core Design Principles

ASP.NET Core was designed around several foundational principles:

1. **Pay-for-what-you-use**: Only the middleware and services you explicitly add are included in the pipeline. No bloated default stack.
2. **Dependency injection first**: DI is built into the framework at every level, not bolted on as an afterthought.
3. **Configuration flexibility**: Multiple configuration sources (JSON, environment variables, command line, Azure Key Vault, user secrets) are unified under a single API.
4. **Testability**: Every component can be mocked and tested in isolation thanks to interface-driven design.
5. **Performance**: ASP.NET Core consistently ranks among the fastest web frameworks in the TechEmpower benchmarks.

> [!summary] Section Summary
> - ASP.NET Core is a cross-platform, high-performance web framework running on .NET
> - It is modular, testable, and built with dependency injection at its core
> - It supports Minimal APIs (from .NET 6+) for lightweight endpoint definitions
> - It follows a pay-for-what-you-use model -- no unnecessary defaults in the pipeline
