---
tags: [csharp, asp-net-core, project-structure]
---


> [!tip] Complete Summary
> An ASP.NET Core project is built from a well-defined set of files and folders, each serving a distinct purpose:
>
> - **`.csproj`** -- The project file defines the SDK (`Microsoft.NET.Sdk.Web` for web apps), target framework, nullable settings, and NuGet package references. It is the single source of truth for how the project is built.
> - **`Program.cs`** -- The sole entry point since .NET 6. It has two phases: the builder phase (registering services into the DI container) and the app phase (configuring the middleware pipeline). Middleware order matters critically.
> - **`appsettings.json`** and **`appsettings.{Environment}.json`** -- Layered configuration files. Environment-specific files override the base. Secrets should never be stored here. Use the Options pattern (`IOptions<T>`) for strongly-typed access.
> - **`Properties/launchSettings.json`** -- Controls local development launch behavior: URLs, ports, environment variables, and browser launch. Not used in production.
> - **`wwwroot/`** -- The web root for static files (CSS, JS, images). Files here are publicly accessible; files outside are protected. Enabled by `UseStaticFiles()` middleware.
> - **`Controllers/`** -- Contains controller classes for MVC and Web API projects. API controllers inherit `ControllerBase`; MVC controllers inherit `Controller`.
> - **`Models/`** -- Holds entities, view models, DTOs, and enums that represent application data.
> - **`Views/`** (MVC) -- Razor `.cshtml` files organized by controller name, with shared layouts and imports.
> - **`Pages/`** (Razor Pages) -- Page-focused structure where each `.cshtml` view is paired with a `.cshtml.cs` PageModel. Routing is file-based.
> - **SDK choice** -- `Microsoft.NET.Sdk.Web` for web apps (includes ASP.NET shared framework), `Microsoft.NET.Sdk` for libraries and console apps.
> - **`dotnet new` templates** -- Scaffold projects from the CLI using templates like `webapp`, `webapi`, `mvc`, `web`, `blazor`, `worker`, and more.
>
> Understanding this structure is the foundation for everything else in ASP.NET Core development -- dependency injection, middleware, routing, and deployment all build on top of these conventions.
