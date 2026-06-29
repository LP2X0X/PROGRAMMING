---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - fundamentals
---

## Comprehensive Summary

> [!tip] Complete Summary
> **What DI is:** A design pattern where classes receive dependencies from the outside rather than creating them with `new`. It implements the Dependency Inversion Principle -- depend on abstractions, not concretions.
>
> **Why it matters:** Without DI, code is tightly coupled, difficult to test, and resistant to change. With DI, you can swap implementations, mock dependencies in tests, and see all dependencies at a glance in the constructor.
>
> **How ASP.NET Core implements it:** The framework provides a built-in DI container with two key interfaces -- `IServiceCollection` for registration and `IServiceProvider` for resolution. You register services in `Program.cs` via `builder.Services`, and the container automatically resolves the full dependency graph when creating controllers and other framework-managed objects.
>
> **Three registration approaches:** Interface-based (`AddScoped<IService, Impl>()`) for most services, self-registration (`AddScoped<ConcreteClass>()`) for simple cases, and factory-based (`AddScoped<IService>(sp => ...)`) for complex construction logic.
>
> **Constructor injection** is the primary pattern -- declare dependencies as constructor parameters, store them in `readonly` fields, and let the framework handle the rest. The next topics to explore are [[Service Lifetimes]] (Scoped, Singleton, Transient), [[Registration Patterns]], and [[Common DI Pitfalls]].

---

## Related Topics

- [[Service Lifetimes]] -- Scoped vs Singleton vs Transient and when to use each
- [[Registration Patterns]] -- Advanced registration techniques including keyed services, open generics, and decorator patterns
- [[Common DI Pitfalls]] -- Captive dependencies, service locator anti-pattern, and other mistakes to avoid
- [[Middleware in ASP.NET Core]] -- How middleware interacts with DI
- [[Options Pattern]] -- Using `IOptions<T>` for strongly-typed configuration
- [[Unit Testing with DI]] -- Mocking strategies and test service providers

---

## Further Reading

- [Microsoft Docs: Dependency Injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)
- [Microsoft Docs: Dependency Inversion Principle](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles#dependency-inversion)
- [Microsoft Docs: .NET Dependency Injection Guidelines](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines)
- [Mark Seemann -- Dependency Injection Principles, Practices, and Patterns (book)](https://www.manning.com/books/dependency-injection-principles-practices-and-patterns)
