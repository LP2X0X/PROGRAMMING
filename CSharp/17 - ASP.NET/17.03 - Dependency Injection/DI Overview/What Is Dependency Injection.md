---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - fundamentals
---

## What Is Dependency Injection

> [!info] Definition
> **Dependency Injection (DI)** is a design pattern where an object receives the other objects it depends on (its *dependencies*) from an external source, rather than creating them itself. The "injection" typically happens through the constructor, though method and property injection also exist.

DI is a specific technique for achieving a broader principle from SOLID: the **Dependency Inversion Principle (DIP)**.

The Dependency Inversion Principle states:

1. **High-level modules should not depend on low-level modules.** Both should depend on abstractions (interfaces or abstract classes).
2. **Abstractions should not depend on details.** Details (concrete implementations) should depend on abstractions.

In practice, this means your `OrderService` should not directly reference a concrete `SqlInventoryRepository`. Instead, it should depend on an `IInventoryRepository` interface. The concrete implementation is decided elsewhere -- typically in the DI container configuration -- and *injected* at runtime.

```csharp
// The high-level module depends on an abstraction, not a concrete class
public class OrderService
{
    private readonly IInventoryRepository _inventory;

    // The dependency is injected -- OrderService does not decide which implementation to use
    public OrderService(IInventoryRepository inventory)
    {
        _inventory = inventory;
    }
}
```

> [!ad-note]
> DI is not unique to ASP.NET Core. It is a general object-oriented design pattern used across languages and frameworks. What ASP.NET Core provides is a *built-in DI container* that automates the process of creating objects and injecting their dependencies. In desktop .NET development, you may have used DI manually or through third-party containers like Autofac or Ninject. ASP.NET Core makes DI a first-class citizen that you interact with on every project.

> [!summary] Section Summary
> - Dependency Injection is a pattern where dependencies are provided to a class from the outside, not created internally.
> - It implements the Dependency Inversion Principle: depend on abstractions (interfaces), not concrete implementations.
> - High-level business logic classes should never directly instantiate their low-level dependencies.
> - ASP.NET Core ships with a built-in DI container, making DI a core part of the framework rather than an optional add-on.
