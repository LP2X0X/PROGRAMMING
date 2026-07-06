---
tags: [csharp, asp-net-core, fundamentals, web]
---


ASP.NET Core does not exist in isolation. It sits within a broader ecosystem of libraries and frameworks:

```
+-----------------------------------------------------+
|                Your ASP.NET Core App                 |
|  (Controllers, Minimal APIs, Blazor, Razor Pages)    |
+-----------------------------------------------------+
|                ASP.NET Core Framework                |
|  (Routing, Middleware, Auth, Model Binding, Filters)  |
+-----------------------------------------------------+
|              .NET Base Class Libraries                |
|  (Collections, IO, Networking, JSON, Cryptography)    |
+-----------------------------------------------------+
|                  .NET Runtime (CLR)                   |
|  (GC, JIT, Type System, Threading)                    |
+-----------------------------------------------------+
|                Operating System                       |
|  (Windows, Linux, macOS)                              |
+-----------------------------------------------------+
```

### Key Companion Libraries

| Library | Relationship |
|---|---|
| [[DbContext\|Entity Framework Core]] | ORM for database access; integrates via DI and `DbContext` |
| [[17.03 - Dependency Injection\|DI Container]] | Built into ASP.NET Core; resolves all services |
| [[Hosting Model\|Kestrel / Hosting]] | The HTTP server that powers ASP.NET Core |
| Serilog / NLog | Structured logging providers that plug into the logging abstraction |
| MediatR | Mediator pattern library commonly used with CQRS in ASP.NET Core |
| FluentValidation | Replaces or supplements DataAnnotations for request validation |
| AutoMapper / Mapster | Object-to-object mapping between DTOs and domain models |

> [!example] Typical Service Registration
> A real-world ASP.NET Core `Program.cs` brings together multiple libraries:
> ```csharp
> var builder = WebApplication.CreateBuilder(args);
> 
> // ASP.NET Core
> builder.Services.AddControllers();
> builder.Services.AddEndpointsApiExplorer();
> 
> // Entity Framework Core
> builder.Services.AddDbContext<InventoryContext>(options =>
>     options.UseSqlServer(builder.Configuration.GetConnectionString("InventoryDb")));
> 
> // Application services
> builder.Services.AddScoped<IOrderService, OrderService>();
> builder.Services.AddScoped<IInventoryService, InventoryService>();
> 
> // Third-party: Serilog
> builder.Host.UseSerilog((context, config) =>
>     config.ReadFrom.Configuration(context.Configuration));
> 
> var app = builder.Build();
> 
> app.UseSerilogRequestLogging();
> app.UseHttpsRedirection();
> app.UseAuthorization();
> app.MapControllers();
> 
> app.Run();
> ```

> [!summary] Section Summary
> - ASP.NET Core sits on top of the .NET BCL and CLR runtime
> - It integrates tightly with EF Core for data access and the built-in DI container
> - Third-party libraries (Serilog, MediatR, FluentValidation) plug in through standardized abstractions
> - A typical `Program.cs` composes ASP.NET Core, EF Core, and application services together
