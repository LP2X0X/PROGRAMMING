---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


Choosing between minimal APIs and [[API Controllers]] depends on the project's size, team, and requirements.

### Feature Comparison

| Feature                       | Minimal APIs                      | Controllers                          |
|---|---|---|
| Startup code                  | Very little                       | More boilerplate                     |
| File organization             | Flexible (any file)               | Convention-driven (Controllers/)     |
| Routing                       | Fluent API (`MapGet`, etc.)       | Attribute or conventional routing    |
| Model validation              | Manual / endpoint filters         | Automatic with `[ApiController]`     |
| Action filters                | Endpoint filters (.NET 7+)        | Full filter pipeline (5 types)       |
| Model binding                 | Automatic + attributes            | Automatic + attributes               |
| Content negotiation           | Supported                         | Full support with formatters         |
| API versioning                | Via packages/groups               | Built-in conventions                 |
| OpenAPI                       | Metadata methods / TypedResults   | Attribute-based / conventions        |
| Dependency injection          | Handler parameters                | Constructor injection                |
| Testability                   | Good (TypedResults)               | Good (ActionResult<T>)              |
| `IActionFilter` / `IAsyncActionFilter` | Not available             | Full support                         |
| `OutputFormatter` negotiation | Limited                           | Full pipeline                        |
| OData support                 | Not supported                     | Supported                            |

### When to Use Minimal APIs

- **Microservices** with a small number of focused endpoints
- **Prototyping** and proof-of-concept applications
- **Serverless functions** (Azure Functions-style handlers)
- **Small APIs** (under ~20-30 endpoints)
- **BFF (Backend-for-Frontend)** layers that proxy to other services
- When you want **maximum control** with minimum ceremony
- **Learning ASP.NET Core** -- lower barrier to entry

### When to Use Controllers

- **Large APIs** with dozens or hundreds of endpoints
- When you need the **full filter pipeline** (authorization, resource, action, result, exception filters)
- When you rely on **OData** for queryable endpoints
- When your team is **familiar with MVC conventions**
- When you need **complex content negotiation** with multiple output formatters
- **Enterprise applications** where convention-over-configuration is preferred

### Hybrid Approach

You can use both in the same application:

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers(); // Add controller support

var app = builder.Build();

// Minimal API endpoints
app.MapGet("/health", () => Results.Ok("Healthy"));
app.MapProductEndpoints(); // Minimal API group

// Controller endpoints
app.MapControllers(); // Map attribute-routed controllers

app.Run();
```

> [!example]
> A common pattern: use minimal APIs for simple health checks, configuration endpoints, and lightweight microservice routes, while using controllers for the main business API that benefits from filters and conventions.

> [!summary] Section Summary
> Minimal APIs excel at small, focused APIs, microservices, prototyping, and serverless scenarios. Controllers are better for large APIs needing the full filter pipeline, OData, or complex content negotiation. Both can coexist in the same application. Choose based on the project's complexity and team familiarity.
