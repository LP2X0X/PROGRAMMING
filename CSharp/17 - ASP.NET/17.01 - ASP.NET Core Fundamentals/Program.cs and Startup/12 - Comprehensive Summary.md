---
tags: [csharp, asp-net-core, startup, program]
---


> [!tip] Complete Summary
> **Program.cs** is the single entry point for modern ASP.NET Core applications. It replaces the older two-file model (`Program.cs` + `Startup.cs`) with a streamlined approach using top-level statements and the minimal hosting API.
>
> The application lifecycle follows a **two-phase pattern**:
> 1. **Builder Phase** — `WebApplication.CreateBuilder(args)` sets up configuration, logging, DI, and Kestrel. You register all services via `builder.Services`, access configuration via `builder.Configuration`, and customize logging via `builder.Logging`.
> 2. **App Phase** — `builder.Build()` freezes the DI container and returns a `WebApplication`. You then configure the middleware pipeline with `app.Use*()` calls (order matters critically) and map endpoints with `app.Map*()` calls.
>
> `app.Run()` starts Kestrel and blocks until the application shuts down gracefully.
>
> Key rules to remember:
> - All service registration happens **before** `Build()`.
> - Middleware order determines request handling behavior: exception handling first, endpoints last, authentication before authorization.
> - `CreateBuilder` provides sensible defaults (Kestrel, layered configuration, console logging) that cover most scenarios.
> - The old `Startup.cs` pattern is still supported but the community has standardized on the minimal hosting model.
> - For integration tests, add `public partial class Program { }` at the bottom of `Program.cs`.
