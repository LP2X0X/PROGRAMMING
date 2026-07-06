---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - pitfalls
---

## Comprehensive Summary

> [!tip] Complete Summary
> The most critical DI pitfalls in ASP.NET Core, ranked by frequency and severity:
>
> **Captive Dependency** -- A Singleton captures a Scoped service, causing stale data, memory leaks, and thread-safety violations. Fix with `IServiceScopeFactory`. Enable `ValidateScopes` to catch it early.
>
> **Service Locator** -- Injecting `IServiceProvider` and calling `GetService<T>()` hides dependencies, complicates testing, and removes compile-time safety. Use constructor injection for known types; reserve `IServiceProvider` for runtime-determined types and infrastructure code.
>
> **Constructor Over-Injection** -- 7+ constructor parameters indicates SRP violation. Refactor by introducing facade services, splitting classes, or using the Mediator pattern.
>
> **Disposable Mismanagement** -- Never call `Dispose()` on container-managed services. The container handles disposal for services it creates. You only manage disposal for objects you create with `new`.
>
> **Root Scope Resolution** -- Resolving Scoped services from `app.Services` or middleware constructors creates root-scoped instances. Use `CreateScope()` for startup code and `HttpContext.RequestServices` or method injection for middleware.
>
> **Circular Dependencies** -- A depends on B depends on A causes resolution failure. Fix by restructuring, extracting shared logic, or using `Lazy<T>` as a temporary measure.
>
> **Singleton Thread Safety** -- Singletons are shared across all threads. Protect mutable state with `ConcurrentDictionary`, `lock`, or `Interlocked`. Prefer stateless Singletons.
>
> **Missing Registrations** -- "Unable to resolve service" means a registration is missing, uses the wrong interface, or has a namespace mismatch. Enable `ValidateOnBuild` to catch these at startup.
>
> **Using `new` for Services** -- Manually newing up service dependencies bypasses DI entirely. The created instance gets none of its own dependencies. Always inject through the constructor.
>
> **Golden Rule**: When in doubt, enable both `ValidateScopes` and `ValidateOnBuild` in all environments. The small performance overhead is insignificant compared to the hours spent debugging captive dependencies or missing registrations in production.

---

## Related Topics

- [[DI Overview]] -- Fundamentals of dependency injection in ASP.NET Core
- [[Service Lifetimes]] -- Transient, Scoped, and Singleton lifetime behavior in detail
- [[Registration Patterns]] -- How to register services (AddTransient, AddScoped, AddSingleton, factory delegates, assembly scanning)
- [[IHostedService and Background Services]] -- Background services where captive dependency and scoping pitfalls are most common
- [[Middleware Pipeline]] -- How middleware interacts with DI scopes
- [[Entity Framework Core DbContext]] -- DbContext lifecycle and why it must remain Scoped
- [[Unit Testing with Mocks]] -- Why proper DI matters for testability

---

## Further Reading

- Microsoft Docs: Dependency Injection in ASP.NET Core -- https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
- Microsoft Docs: Dependency Injection Guidelines -- https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines
- Mark Seemann, "Dependency Injection Principles, Practices, and Patterns" (Manning, 2nd Edition) -- the definitive book on DI in .NET
- Steve Smith (Ardalis): Avoiding Captive Dependencies -- https://ardalis.com/avoid-captive-dependencies/
- Andrew Lock: Exploring the .NET Core Dependency Injection Container -- https://andrewlock.net/exploring-the-dotnet-core-dependency-injection-container/
