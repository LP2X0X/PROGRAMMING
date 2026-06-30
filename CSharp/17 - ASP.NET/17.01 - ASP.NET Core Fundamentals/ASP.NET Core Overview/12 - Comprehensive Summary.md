---
tags: [csharp, asp-net-core, fundamentals, web]
---


> [!tip] Complete Summary
> **ASP.NET Core** is a cross-platform, high-performance, open-source web framework that is a **complete ground-up rewrite** of the legacy ASP.NET Framework. It runs on .NET 6/8/9+ and supports Windows, Linux, and macOS.
>
> **What you can build**: MVC web apps, REST APIs, Razor Pages, Blazor interactive UIs, gRPC services, and SignalR real-time apps -- all under one unified framework.
>
> **Key design principles**: modular middleware pipeline (pay-for-what-you-use), built-in dependency injection, unified configuration system (JSON + env vars + Options pattern), and testability through interface-driven design.
>
> **Performance**: ASP.NET Core ranks among the fastest web frameworks globally, powered by the Kestrel HTTP server, async I/O, `Span<T>`/`Memory<T>` optimizations, and support for HTTP/2, HTTP/3, and Native AOT.
>
> **Ecosystem**: The `Microsoft.AspNetCore.App` shared framework is included implicitly via the `Microsoft.NET.Sdk.Web` SDK. Additional libraries like EF Core, JWT authentication, and structured logging are added as separate NuGet packages.
>
> **Release cadence**: Even-numbered releases (.NET 8, 10) are LTS with 3-year support; odd-numbered releases (.NET 9) are STS with 18-month support. Always prefer the latest LTS for production.
>
> **Migration note**: ASP.NET Framework is in maintenance mode. All new projects should use ASP.NET Core. Legacy technologies like Web Forms and WCF Server have no direct equivalent -- use Blazor/Razor Pages and gRPC respectively.
>
> **Next steps**: Explore [[Project Structure]] for how ASP.NET Core apps are organized, [[Hosting Model]] for Kestrel and reverse proxy configuration, [[Program.cs and Startup]] for the application bootstrap process, and [[Environments]] for managing Development/Staging/Production settings.
