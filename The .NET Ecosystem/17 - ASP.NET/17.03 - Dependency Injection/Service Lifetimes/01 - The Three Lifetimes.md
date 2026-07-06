---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - lifetimes
---

## The Three Lifetimes

When you register a service with the ASP.NET Core DI container, you must choose one of three lifetimes. Each lifetime answers a single question: **when does the container create a new instance, and when does it reuse an existing one?**

Think of it with this mental model:

| Lifetime | Mental Model |
|---|---|
| **Transient** | A vending machine -- every button press gives you a fresh item, even if you press the same button twice in a row |
| **Scoped** | A whiteboard in a meeting room -- everyone in the same meeting (request) shares it, but the next meeting gets a clean board |
| **Singleton** | A wall clock in the office -- there is exactly one, everyone sees the same clock, it exists for the life of the building |

All three are registered through `IServiceCollection` in `Program.cs` (or `Startup.cs` in older project styles):

```csharp
var builder = WebApplication.CreateBuilder(args);

// Transient -- new instance every time
builder.Services.AddTransient<IOrderValidator, OrderValidator>();

// Scoped -- one instance per HTTP request
builder.Services.AddScoped<IOrderRepository, OrderRepository>();

// Singleton -- one instance for the entire app
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
```

> [!info] Definition
> A **service lifetime** governs how the DI container creates, reuses, and disposes of service instances. It is set at registration time and cannot be changed at runtime.

> [!summary] Section Summary
> - ASP.NET Core offers three service lifetimes: Transient, Scoped, and Singleton.
> - The lifetime is chosen at registration time via `AddTransient`, `AddScoped`, or `AddSingleton`.
> - Each lifetime controls instance creation, reuse, and disposal differently.
> - Choosing the wrong lifetime leads to bugs around stale data, thread safety, and memory leaks.
> - The mental models (vending machine, whiteboard, wall clock) help anchor the differences.
