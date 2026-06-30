---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


### Naming Convention

Partial views are prefixed with an underscore (`_`). This is a **convention**, not a requirement, but it serves two purposes:
1. Visually distinguishes partials from full views in the file explorer
2. Prevents partials from being returned directly by a controller (convention-based routing skips `_`-prefixed files)

### Discovery Order

When you reference `<partial name="_ProductCard" />`, Razor searches in this order:

1. **Same folder** as the calling view (e.g., `/Views/Products/`)
2. **`/Views/Shared/`**
3. **`/Pages/Shared/`** (if using [[Razor Pages]])

If you need a partial from a specific location, use a full path:

```cshtml
<partial name="/Views/Admin/Shared/_AdminWidget.cshtml" />
```

> [!ad-note] Area Partials
> In area-based applications, the search also includes the area's `Shared` folder:
> `/Areas/Admin/Views/Shared/_AdminWidget.cshtml`

> [!summary] Section Summary
> - Partial views use an underscore prefix by convention (`_PartialName.cshtml`)
> - Razor searches the current view's folder first, then `/Views/Shared/`, then `/Pages/Shared/`
> - Use full paths when the default discovery order does not find the right partial

---
