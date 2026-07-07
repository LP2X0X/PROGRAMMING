---
tags:
  - uml
  - package-diagram
  - structural
---

## 🔹 What It Shows

- A **package diagram** shows how model elements (classes, interfaces, other packages) are organized into **packages** — logical groupings that act as namespaces.
- Packages are containers. They group related elements and define **dependency relationships** between those groups.
- Maps directly to **namespaces** in C# (`namespace MyApp.Data`), **packages** in Java (`package com.myapp.data`), and **modules** in Python.

**When to use:**

- Large systems where you need a bird's-eye view of structure
- Visualizing and enforcing **dependency rules** between layers or modules
- Communicating project organization to new team members
- Identifying **coupling problems** before they get expensive to fix
- Planning how to split a monolith or organize a new solution

---

## 🔹 Quick Reference

| Element              | Notation                                  | Description                                                  |
| -------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| **Package**          | Folder/tab shape                          | A namespace or logical grouping of elements                  |
| **Dependency**       | `- - - ->` dashed arrow                   | Source package depends on target package                      |
| **`<<import>>`**     | `- - <<import>> - ->` dashed arrow        | Public import — elements become visible in source namespace  |
| **`<<access>>`**     | `- - <<access>> - ->` dashed arrow        | Private import — elements available but not re-exported      |
| **`<<merge>>`**      | `- - <<merge>> - ->` dashed arrow         | Combine package contents (rarely used)                       |
| **Nested package**   | Package drawn inside another package      | Sub-namespace (e.g., `MyApp.Data.Repositories`)              |
| **Public element**   | `+ClassName`                              | Visible outside the package                                  |
| **Private element**  | `-ClassName`                              | Internal to the package, not exported                        |

---

## 🔹 Package Notation

A package is drawn as a **folder with a tab**. The name goes in the tab or in the body.

### With contents visible

```
┌──────────┐
│  Utils   │
├────────────────────┐
│                    │
│  + StringHelper    │
│  + DateHelper      │
│  - InternalCache   │
│                    │
└────────────────────┘
```

### Name only (contents hidden)

```
┌──────────┐
│  Utils   │
├────────────────────┐
│                    │
│                    │
└────────────────────┘
```

- A package can contain: **classes**, **interfaces**, **enums**, **other packages**, or any UML element.
- Use the "contents visible" form when you want to show what lives inside. Use "name only" when you are focusing on inter-package dependencies and the internals do not matter.

---

## 🔹 Package Dependencies

Dependencies between packages are shown with **dashed arrows**. The arrow points from the **dependent** (source) to the **dependency** (target): "A depends on B" means A uses something from B.

### `<<import>>` — Public Import

The source package gains public access to the target's exported elements. Any package that later imports the source also sees these elements (they are **re-exported**).

```
┌──────────┐                    ┌──────────┐
│  WebApp  │                    │  Models  │
├──────────────┐                ├──────────────┐
│              │                │              │
│              │╌╌╌╌╌╌╌╌╌╌╌╌╌╌>│              │
│              │  <<import>>    │              │
│              │                │              │
└──────────────┘                └──────────────┘
```

In C# terms: this is like adding `using MyApp.Models;` — the types become available, and anything that references `WebApp` can also see them.

### `<<access>>` — Private Import

Same as import, but the elements are **not re-exported**. Only the source package can use them internally.

```
┌──────────┐                    ┌──────────┐
│  Orders  │                    │  Logging │
├──────────────┐                ├──────────────┐
│              │                │              │
│              │╌╌╌╌╌╌╌╌╌╌╌╌╌╌>│              │
│              │  <<access>>    │              │
│              │                │              │
└──────────────┘                └──────────────┘
```

In C# terms: think of this as an `internal` dependency — `Orders` uses `Logging` internally, but consumers of `Orders` do not gain access to `Logging`.

### `<<merge>>` — Package Merge

Combines the contents of the target package into the source. Elements with the same name are merged together. This is rarely used in practice — mostly seen in UML metamodel specifications.

```
┌──────────┐                    ┌──────────┐
│ Extended │                    │   Base   │
├──────────────┐                ├──────────────┐
│              │                │              │
│              │╌╌╌╌╌╌╌╌╌╌╌╌╌╌>│              │
│              │  <<merge>>     │              │
│              │                │              │
└──────────────┘                └──────────────┘
```

### Plain Dependency (no stereotype)

When you just want to show "A depends on B" without specifying the mechanism:

```
┌──────────┐                    ┌──────────┐
│    UI    │                    │   Core   │
├──────────────┐                ├──────────────┐
│              │                │              │
│              │╌╌╌╌╌╌╌╌╌╌╌╌╌╌>│              │
│              │                │              │
└──────────────┘                └──────────────┘
```

```ad-info
Arrow direction matters: the arrow always points **toward the package being depended on**. Read it as "A uses B" or "A depends on B". The dependent is the tail, the dependency is the head.
```

---

## 🔹 Nested Packages

Packages can be **nested** inside other packages to represent sub-namespaces or sub-modules.

```
┌────────────┐
│  MyApp     │
├────────────────────────────────────────┐
│                                        │
│  ┌──────────┐       ┌──────────┐      │
│  │  Data    │       │  Core    │      │
│  ├──────────────┐   ├──────────────┐  │
│  │              │   │              │  │
│  │ Repository   │   │  IService    │  │
│  │ DbContext    │   │  BaseEntity  │  │
│  │              │   │              │  │
│  └──────────────┘   └──────────────┘  │
│                                        │
└────────────────────────────────────────┘
```

This corresponds directly to nested namespaces in C#:

```csharp
namespace MyApp.Data
{
    public class Repository { }
    public class DbContext { }
}

namespace MyApp.Core
{
    public interface IService { }
    public class BaseEntity { }
}
```

- Nesting implies **containment**, not dependency. A nested package belongs to its parent, but you still need explicit dependency arrows to show usage relationships.
- Keep nesting to 2-3 levels maximum — deeper nesting becomes unreadable.

---

## 🔹 Package Visibility

Elements inside a package have **visibility** that controls whether they are accessible from outside.

| Visibility | Symbol | Meaning                                      | C# Equivalent           |
| ---------- | ------ | -------------------------------------------- | ------------------------ |
| Public     | `+`    | Accessible to any package that imports this  | `public` class/interface |
| Private    | `-`    | Only accessible within the owning package    | `internal` class         |

```
┌────────────┐
│  Services  │
├─────────────────────┐
│                     │
│  + OrderService     │  ← visible to importers
│  + PaymentService   │  ← visible to importers
│  - OrderValidator   │  ← internal helper, hidden
│  - PriceCalculator  │  ← internal helper, hidden
│                     │
└─────────────────────┘
```

```ad-tip
Use visibility to communicate your **public API surface**. Mark implementation details as private (`-`) to signal that other packages should not depend on them. This maps to the `internal` access modifier in C# — types that help your package do its job but are not part of the contract.
```

---

## 🔹 Real-World Examples

### Example 1: Layered Architecture

A classic three-tier architecture with dependencies flowing **downward only**.

```
┌──────────────┐
│ Presentation │
├──────────────────────────┐
│                          │
│  + HomeController        │
│  + OrderController       │
│  + Views                 │
│                          │
└──────────┬───────────────┘
           │
           │ <<import>>
           v
┌──────────────────┐
│  Business Logic  │
├──────────────────────────┐
│                          │
│  + OrderService          │
│  + PaymentService        │
│  - OrderValidator        │
│                          │
└──────────┬───────────────┘
           │
           │ <<import>>
           v
┌──────────────┐
│ Data Access  │
├──────────────────────────┐
│                          │
│  + OrderRepository       │
│  + ProductRepository     │
│  - QueryBuilder          │
│                          │
└──────────┬───────────────┘
           │
           │ <<access>>
           v
┌──────────────┐
│   Database   │
├──────────────────────────┐
│                          │
│  + DbContext             │
│  + ConnectionFactory     │
│                          │
└──────────────────────────┘
```

Key observations:
- All arrows point **downward** — upper layers depend on lower layers, never the reverse
- `Presentation` knows about `Business Logic` but not `Data Access` directly
- `Database` uses `<<access>>` because the connection details are an internal concern of `Data Access` — not re-exported to higher layers

### Example 2: .NET Namespace Organization

A typical C#/.NET solution structure modeled as packages.

```
┌─────────────┐
│  MyApp.Web  │
├─────────────────────────┐
│                         │
│  + Controllers/         │
│  + ViewModels/          │
│  + Middleware/          │
│                         │
└─────┬───────────────────┘
      │
      │ <<import>>
      v
┌─────────────┐            ┌───────────────┐
│ MyApp.Core  │            │ MyApp.Common  │
├─────────────────────┐    ├───────────────────────┐
│                     │    │                       │
│  + IOrderService    │    │  + StringExtensions   │
│  + IRepository<T>   │    │  + DateTimeHelper     │
│  + Order            │    │  + Result<T>          │
│  + Product          │    │                       │
│                     │    └───────────────────────┘
└─────┬───────────────┘              ^
      │                              │
      │ <<import>>          <<import>> (used by all)
      v                              │
┌─────────────┐                      │
│ MyApp.Data  ├──────────────────────┘
├─────────────────────┐
│                     │
│  + OrderRepository  │
│  + AppDbContext      │
│  - EfQueryHelper    │
│                     │
└─────────────────────┘
```

Dependency flow in this .NET solution:
- `MyApp.Web` references `MyApp.Core` (to use interfaces and models)
- `MyApp.Core` has **no dependencies** on other app packages — it defines contracts only
- `MyApp.Data` references `MyApp.Core` (to implement `IRepository<T>`) and `MyApp.Common` (for utilities)
- `MyApp.Common` has **no dependencies** — it is a leaf package with pure utility code

```ad-info
In a .NET solution, these packages map to **separate projects** (`.csproj` files). Project references in the `.csproj` correspond exactly to the `<<import>>` arrows in the diagram. This is why package diagrams are so useful for .NET developers — they are a visual representation of your solution's project reference graph.
```

### Example 3: Dependency Violation

Here is an architecture with a **circular dependency** — a common mistake that package diagrams help you catch early.

```
┌──────────────┐
│ Presentation │
├──────────────────────────┐
│                          │
│  + OrderController       │
│                          │
└────┬─────────────────────┘
     │
     │ <<import>>
     v
┌──────────────────┐
│  Business Logic  │
├──────────────────────────┐
│                          │
│  + OrderService          │
│                          │
└────┬─────────────────────┘
     │               ^
     │ <<import>>    │ <<import>>    ← VIOLATION
     v               │
┌──────────────┐     │
│ Data Access  ├─────┘
├──────────────────────────┐
│                          │
│  + OrderRepository       │
│                          │
└──────────────────────────┘
```

```ad-warning
`Data Access` imports `Business Logic` — this creates a **circular dependency** (`Business Logic` -> `Data Access` -> `Business Logic`). In C#/.NET, this means neither project can compile without the other, and the build will fail with a circular reference error.

**How to fix it:** extract the shared types (interfaces, DTOs) into a separate package that both can depend on. This is why `MyApp.Core` exists in Example 2 — it breaks the cycle by holding the contracts that both layers reference.
```

**Fixed version:**

```
┌──────────────────┐       ┌──────────────┐
│  Business Logic  │       │     Core     │
├──────────────────────┐   ├──────────────────────┐
│                      │   │                      │
│  + OrderService      │──>│  + IOrderRepository  │
│                      │   │  + OrderDto           │
└──────────────────────┘   │                      │
                           └──────────────────────┘
                                     ^
┌──────────────┐                     │
│ Data Access  │                     │
├──────────────────────┐             │
│                      │─────────────┘
│  + OrderRepository   │  <<import>>
│                      │
└──────────────────────┘
```

Now both `Business Logic` and `Data Access` depend on `Core`, but not on each other. The cycle is broken.

---

## 🔹 Tips

- **Model package-level dependencies, not class-level.** A package diagram shows the big picture. If you need to show individual class relationships, use a [[Class Diagram]] instead.
- **No circular dependencies.** If you draw the diagram and find arrows going both ways between two packages, refactor by extracting shared abstractions into a common package.
- **Keep packages cohesive.** Each package should group elements that change together and serve a single purpose. If a package has classes that belong to unrelated features, split it.
- **Use package diagrams to onboard new developers.** Before showing code, show the package diagram — it communicates the system's structure in minutes.
- **Validate against your build system.** In .NET, your project references should match the dependency arrows exactly. If they diverge, the diagram or the code is wrong.
- **Limit what you show.** A package diagram with 20+ packages and arrows everywhere communicates nothing. Focus on the top-level groupings and their key dependencies. Use separate diagrams to zoom into subsystems.

---

See also: [[Class Diagram]], [[Component Diagram]], [[Deployment Diagram]]
