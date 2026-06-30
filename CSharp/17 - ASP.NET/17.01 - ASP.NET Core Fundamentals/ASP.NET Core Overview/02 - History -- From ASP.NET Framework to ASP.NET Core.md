---
tags: [csharp, asp-net-core, fundamentals, web]
---


> [!warning] Common Misconception
> ASP.NET Core is **NOT** an upgrade or evolution of ASP.NET Framework. It is a **complete ground-up rewrite**. Code from ASP.NET Framework (WebForms, classic MVC with `System.Web`) does not directly port to ASP.NET Core.

### The Legacy: ASP.NET on .NET Framework

ASP.NET was introduced in 2002 as part of the .NET Framework. Over the years it grew to include:

- **Web Forms** (2002) -- event-driven, drag-and-drop web development
- **ASP.NET MVC** (2009) -- Model-View-Controller pattern, more control over HTML
- **ASP.NET Web API** (2012) -- HTTP services for REST APIs
- **SignalR** (2013) -- real-time web communication

All of these were tightly coupled to `System.Web.dll` and IIS on Windows. This coupling meant:

- Windows-only deployment
- Heavy runtime overhead from `System.Web`
- Monolithic framework -- you got everything whether you needed it or not
- Slow iteration cycles tied to .NET Framework releases

### The Rewrite: ASP.NET Core

In 2016, Microsoft released **ASP.NET Core 1.0** alongside **.NET Core 1.0**. The goals were clear:

- Break free from `System.Web` and IIS dependency
- Run on Linux and macOS
- Achieve dramatically better performance
- Enable side-by-side versioning (multiple .NET versions on the same machine)
- Ship as NuGet packages with independent release cycles

### Timeline of Major Releases

| Version | Year | Key Highlights |
|---|---|---|
| ASP.NET Core 1.0 | 2016 | Initial cross-platform release |
| ASP.NET Core 2.0 | 2017 | Razor Pages, metapackage introduced |
| ASP.NET Core 3.0 | 2019 | gRPC support, Blazor Server, dropped .NET Framework target |
| .NET 5 (unified) | 2020 | Merged .NET Core and .NET Framework naming |
| .NET 6 | 2021 | Minimal APIs, Hot Reload (LTS) |
| .NET 7 | 2022 | Rate limiting, output caching, HTTP/3 |
| .NET 8 | 2023 | Native AOT for APIs, Blazor United (LTS) |
| .NET 9 | 2024 | HybridCache, OpenAPI built-in, perf improvements |

> [!ad-note] Naming Clarification
> After .NET Core 3.1, Microsoft dropped "Core" from the product name. **.NET 5** and later are simply called ".NET" -- but the web framework is still commonly referred to as "ASP.NET Core" to distinguish it from the legacy ASP.NET Framework.

> [!summary] Section Summary
> - ASP.NET Core is a ground-up rewrite, not an upgrade of ASP.NET Framework
> - The legacy framework was tightly coupled to `System.Web.dll` and Windows/IIS
> - ASP.NET Core 1.0 launched in 2016 to break those dependencies
> - After .NET Core 3.1, the naming unified to just ".NET" (5, 6, 7, 8, 9...)
> - Code from ASP.NET Framework (especially WebForms) does not directly port over
