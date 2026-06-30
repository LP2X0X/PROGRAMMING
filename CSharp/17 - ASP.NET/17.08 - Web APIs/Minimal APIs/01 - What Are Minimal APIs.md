---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


**Minimal APIs** were introduced in ==.NET 6== as a simplified approach to building HTTP APIs in ASP.NET Core. Instead of using the traditional controller-based pattern (see [[API Controllers]]), minimal APIs let you define endpoints directly in `Program.cs` (or any file) using concise lambda syntax.

The key idea: ==you map HTTP verbs to route handlers without needing controller classes, action methods, or attributes==. The entire pipeline is configured in one place using a fluent API on the `WebApplication` object.

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello, World!");

app.Run();
```

That is a complete, functional ASP.NET Core web API. No `Startup.cs`, no controller class, no `[ApiController]` attribute.

### Why Minimal APIs Exist

Before .NET 6, even a trivial API required:
- A `Startup` class with `ConfigureServices` and `Configure` methods
- At least one controller class inheriting from `ControllerBase`
- Attribute routing or convention-based routing configuration

Minimal APIs remove that ceremony. They are built on the same ASP.NET Core infrastructure (routing, middleware, DI, etc.) but expose it through a more direct programming model.

> [!ad-note]
> Minimal APIs are not a replacement for controllers. They are an alternative that is better suited for certain scenarios. Both approaches coexist in the same application if needed.

### Evolution Across .NET Versions

| .NET Version | Minimal API Feature                                  |
|---|---|
| .NET 6       | Initial release: `MapGet`, `MapPost`, basic binding  |
| .NET 7       | Endpoint filters, route groups, `TypedResults`       |
| .NET 8       | Form binding, `[AsParameters]`, keyed DI, antiforgery|
| .NET 9       | Built-in OpenAPI document generation, `WithOpenApi()` improvements |

> [!summary] Section Summary
> Minimal APIs are a lightweight alternative to controllers for building HTTP APIs in ASP.NET Core. Introduced in .NET 6, they allow you to define endpoints directly with lambda expressions and have gained significant features in each subsequent release.
