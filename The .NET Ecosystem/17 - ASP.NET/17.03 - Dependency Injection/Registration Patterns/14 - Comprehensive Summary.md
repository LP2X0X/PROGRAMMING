---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - patterns
---

## Comprehensive Summary

> [!tip] Complete Summary
> **Basic registration** (`Add{Lifetime}<TService, TImpl>()`) is the workhorse pattern -- use it for 80% or more of your registrations to map interfaces to concrete types.
>
> **Self-registration** (`Add{Lifetime}<T>()`) registers a concrete class as its own service type, appropriate for internal helpers and coordinators that do not need an abstraction.
>
> **Factory registration** (`Add{Lifetime}<T>(sp => ...)`) gives you full control over construction, needed for conditional logic, manual parameters, or complex initialization. Use `sp.GetRequiredService<T>()` to resolve other dependencies inside the factory.
>
> **Instance registration** (`AddSingleton(instance)`) hands a pre-built object to the container but beware: the container will not dispose it.
>
> **Multiple implementations** of the same interface are consumed via `IEnumerable<T>`, while **keyed services** (.NET 8+) let you resolve a specific implementation by key using `[FromKeyedServices("key")]`.
>
> **TryAdd methods** prevent accidental overwrites and are essential for library authors who need to provide defaults without stomping on application registrations.
>
> **Open generics** (`AddScoped(typeof(IRepo<>), typeof(Repo<>))`) cover all closed forms with one registration, ideal for repositories and handlers.
>
> The **decorator pattern** requires factory delegates (or Scrutor's `Decorate<T1, T2>()`) since the built-in container has no native decorator support.
>
> **Extension methods** (`services.AddFeature()`) are the standard pattern for grouping related registrations and keeping `Program.cs` clean and navigable.
>
> **Assembly scanning** via Scrutor or manual reflection automates bulk registration at the cost of explicitness -- best for large codebases with many similar services.
>
> The **Options pattern** (`Configure<T>`, `IOptions<T>`, `IOptionsSnapshot<T>`, `IOptionsMonitor<T>`) is the standard way to bind configuration to typed classes, with validation and reload support built in.

---

## Related Topics

- [[The .NET Ecosystem/17 - ASP.NET/17.03 - Dependency Injection/DI Overview/DI Overview]] -- foundational concepts of dependency injection in ASP.NET Core
- [[Service Lifetimes]] -- transient vs scoped vs singleton in depth
- [[Common DI Pitfalls]] -- captive dependencies, scope validation, and other traps
- [[Options Pattern]] -- deeper dive into `IOptions<T>` and related interfaces
- [[Middleware Pipeline]] -- how DI interacts with the request pipeline
- [[Program.cs Structure]] -- minimal hosting model and application bootstrapping
- [[Unit Testing with DI]] -- mocking and overriding registrations in tests

---

## Further Reading

- [Microsoft Docs: Dependency Injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)
- [Microsoft Docs: Options Pattern in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options)
- [Microsoft Docs: Keyed Services](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection#keyed-services)
- [Scrutor GitHub Repository](https://github.com/khellang/Scrutor)
- [Andrew Lock: Adding Decorated Classes to the ASP.NET Core DI Container](https://andrewlock.net/adding-decorated-classes-to-the-asp-net-core-di-container-using-scrutor/)
- [Steve Smith: ASP.NET Core Dependency Injection Deep Dive](https://ardalis.com/aspnetcore-dependency-injection/)
