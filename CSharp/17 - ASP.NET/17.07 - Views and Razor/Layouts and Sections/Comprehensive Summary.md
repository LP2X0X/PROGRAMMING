---
tags:
  - csharp
  - asp-net-core
  - razor
  - layouts
  - views
---


> [!tip] Complete Summary
> **Layouts** in ASP.NET Core Razor provide a master page mechanism where shared HTML structure (DOCTYPE, head, nav, footer) is defined once and reused by all views. The layout file (`_Layout.cshtml`, typically in `/Views/Shared/`) contains `@RenderBody()` as the insertion point for child view content.
>
> **Setting the layout** happens in three places: `_ViewStart.cshtml` (default for all views), per-view assignment (`Layout = "_SpecialLayout"`), or conditionally based on route/area. Per-view settings override `_ViewStart.cshtml`. For area-based switching, prefer separate `_ViewStart.cshtml` files in each area's Views folder.
>
> **Sections** (`@section Name { }` in child views, `@await RenderSectionAsync("Name", required: false)` in layouts) allow child views to inject content into specific layout regions beyond the main body. Common sections include `Scripts`, `Styles`, and `Breadcrumbs`. `IsSectionDefined()` enables fallback content when a section is not provided.
>
> **Multiple layouts** serve different application areas (public site, admin, auth). **Nested layouts** create hierarchies where a child layout uses a parent layout, but sections must be explicitly forwarded between levels. **`Layout = null`** gives a view full HTML control with no layout wrapping, useful for error pages and email templates.
>
> All of these mechanisms integrate naturally with [[Razor Syntax]], [[Tag Helpers]], [[Partial Views and View Components]], and the broader ASP.NET Core MVC pipeline.

---
