---
tags:
  - csharp
  - asp-net-core
  - filters
  - controllers
  - cross-cutting
---


Filters execute in a strict, well-defined order. Each filter type has a "before" hook and an "after" hook, forming a layered pipeline around the action method:

```
Authorization Filter
  Resource Filter (before)
    Model Binding
      Action Filter (before)
        --- ACTION EXECUTES ---
      Action Filter (after)
    Exception Filter (if exception thrown)
    Result Filter (before)
      --- RESULT EXECUTES ---
    Result Filter (after)
  Resource Filter (after)
```

The five filter types, in execution order:

| Order | Filter Type | Interface | Purpose |
|---|---|---|---|
| 1 | Authorization | `IAuthorizationFilter` | Gate access, run first |
| 2 | Resource | `IResourceFilter` | Caching, short-circuit before model binding |
| 3 | Action | `IActionFilter` | Inspect/modify arguments, wrap action execution |
| 4 | Exception | `IExceptionFilter` | Handle unhandled exceptions from the action |
| 5 | Result | `IResultFilter` | Wrap action result execution |

```ad-attention
The "after" hooks run in **reverse order** (inside-out). If the action throws, the exception filter runs instead of the normal action-after and result filters. Resource filter's "after" hook always runs last, even after result execution.
```
