---
tags:
  - csharp
  - asp-net-core
  - configuration
  - options-pattern
---


> [!tip] Complete Summary
> The **Options pattern** is the standard way to consume configuration in ASP.NET Core. Here is the complete mental model:
>
> **Core idea**: Bind configuration sections to strongly-typed POCOs with `services.Configure<T>(config.GetSection("Name"))`. This gives you type safety, IntelliSense, validation, and testability.
>
> **Three injection interfaces**:
> - `IOptions<T>` -- Singleton, reads once, simplest choice for static config
> - `IOptionsSnapshot<T>` -- Scoped, re-reads per request, cannot be used in Singletons
> - `IOptionsMonitor<T>` -- Singleton with live reload and `OnChange` callbacks, ideal for background services
>
> **Named options**: Register multiple configurations of the same type with `Configure<T>(name, section)` and access via `.Get(name)`.
>
> **Validation**: Use `ValidateDataAnnotations()`, `.Validate()`, or `IValidateOptions<T>` -- and **always** add `.ValidateOnStart()` to fail fast.
>
> **PostConfigure**: Runs after all `Configure` calls to set defaults or enforce invariants.
>
> **Testing**: Use `Options.Create(new MySettings { ... })` to create `IOptions<T>` wrappers without any configuration infrastructure.
