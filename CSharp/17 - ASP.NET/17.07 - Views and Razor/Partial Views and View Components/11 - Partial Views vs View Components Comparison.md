---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


| Aspect | Partial View | View Component |
|---|---|---|
| **Data source** | Receives data from the parent view | Fetches its own data independently |
| **C# logic** | Minimal (formatting only) | Can contain business/data logic |
| **DI support** | Only via `@inject` (limited) | Full constructor injection |
| **File structure** | Single `.cshtml` file | C# class + `.cshtml` view |
| **Invocation** | `<partial name="..." />` | `<vc:name />` or `Component.InvokeAsync()` |
| **Testability** | Hard to unit test | Easy to unit test (injectable dependencies) |
| **Caching** | No built-in support | Can implement caching in `InvokeAsync()` |
| **Complexity** | Low | Medium |
| **Typical uses** | Cards, form fields, list items | Nav menus, cart widgets, recommendations |
| **Analogy** | A function that formats data | A mini-controller with its own view |

**Decision guide:**
1. Does the fragment need to fetch data the parent does not have? **View Component**.
2. Does the fragment just render data the parent already prepared? **Partial View**.
3. Does the fragment need caching, logging, or complex service calls? **View Component**.
4. Is the fragment a simple HTML template with model binding? **Partial View**.

> [!summary] Section Summary
> - Partial views are simple and lightweight -- use them for rendering pre-prepared data
> - View components are powerful and self-contained -- use them for independent data fetching
> - The decision hinges on whether the fragment needs its own data pipeline
> - View components are more testable due to constructor injection

---
