---
tags:
  - csharp
  - asp-net-core
  - razor
  - layouts
  - views
---


There are three ways to specify which layout a view uses, in order of precedence:

### 1. Per-View (Highest Priority)

Set `Layout` directly in the view's code block:

```cshtml
@{
    Layout = "_SpecialLayout";
}

<h1>This page uses a special layout</h1>
```

### 2. Via _ViewStart.cshtml (Default for All Views)

The most common approach. `_ViewStart.cshtml` runs before every view and sets the default layout:

```cshtml
@* /Views/_ViewStart.cshtml *@
@{
    Layout = "_Layout";
}
```

### 3. Conditional Layout in _ViewStart.cshtml

```cshtml
@* /Views/_ViewStart.cshtml *@
@{
    if (Context.Request.Path.StartsWithSegments("/admin"))
    {
        Layout = "_AdminLayout";
    }
    else
    {
        Layout = "_Layout";
    }
}
```

> [!tip] Practical Tip
> For area-based layout switching, place a separate `_ViewStart.cshtml` in the area's Views folder rather than using conditionals in the root `_ViewStart.cshtml`. This is cleaner and scales better.
>
> ```
> /Views/_ViewStart.cshtml         → Layout = "_Layout"
> /Areas/Admin/Views/_ViewStart.cshtml → Layout = "_AdminLayout"
> ```

**Layout name resolution:**
- If you specify `"_Layout"`, Razor searches:
  1. The same folder as the current view
  2. `/Views/Shared/`
  3. `/Pages/Shared/` (for Razor Pages)
- You can also specify a full path: `"/Views/Shared/_SpecialLayout.cshtml"`

> [!summary] Section Summary
> - Layout can be set per-view, in `_ViewStart.cshtml`, or conditionally
> - Per-view `Layout` assignment overrides `_ViewStart.cshtml`
> - Layout names are resolved by searching the view's folder, then `/Views/Shared/`
> - Area-specific `_ViewStart.cshtml` files provide clean layout switching

---
