---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


Razor Pages use the same [[Layouts and Sections|layout system]], [[Partial Views and View Components|partial views and view components]], and [[Tag Helpers]] as MVC views.

### _ViewStart.cshtml for Razor Pages

```cshtml
@* /Pages/_ViewStart.cshtml *@
@{
    Layout = "_Layout";
}
```

Razor Pages search for layouts in:
1. The same folder as the page
2. `/Pages/Shared/`
3. `/Views/Shared/` (shared with MVC views)

### _ViewImports.cshtml for Razor Pages

```cshtml
@* /Pages/_ViewImports.cshtml *@
@using MyApp
@using MyApp.Models
@namespace MyApp.Pages
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

> [!ad-note] Shared Layouts Between MVC and Razor Pages
> If your application uses both MVC controllers and Razor Pages, a layout in `/Views/Shared/` is accessible to both. You do not need duplicate layouts.

### Partial Views

```cshtml
@* Works identically to MVC *@
<partial name="_ProductCard" model="product" />
```

### View Components

```cshtml
@* Works identically to MVC *@
<vc:cart-summary />
@await Component.InvokeAsync("RecentProducts", new { count = 5 })
```

> [!summary] Section Summary
> - Razor Pages use the same `_ViewStart.cshtml`, `_ViewImports.cshtml`, and layout system as MVC
> - Layouts are searched in the page's folder, `/Pages/Shared/`, and `/Views/Shared/`
> - Partial views and view components work identically to MVC
> - MVC and Razor Pages can share layouts, partials, and view components

---
