---
tags: [csharp, asp-net-core, project-structure]
---


The `Views/` folder exists in MVC projects and contains `.cshtml` Razor files that define the HTML rendered to the browser. Views are organized by controller name.

### Folder Structure

```
Views/
  Orders/
    Index.cshtml
    Details.cshtml
    Create.cshtml
  Shared/
    _Layout.cshtml
    _Layout.cshtml.css
    _ValidationScriptsPartial.cshtml
    Error.cshtml
  _ViewImports.cshtml
  _ViewStart.cshtml
```

### Convention: Controller-to-View Mapping

When a controller action calls `return View()`, ASP.NET Core looks for a matching `.cshtml` file by convention:

| Controller | Action | View Path |
|---|---|---|
| `OrdersController` | `Index()` | `Views/Orders/Index.cshtml` |
| `OrdersController` | `Details(int id)` | `Views/Orders/Details.cshtml` |
| `ProductsController` | `Create()` | `Views/Products/Create.cshtml` |

### Special Files

| File | Purpose |
|---|---|
| `_Layout.cshtml` | The master layout template (header, footer, nav) |
| `_ViewStart.cshtml` | Runs before every view; typically sets the layout |
| `_ViewImports.cshtml` | Declares shared `@using` and `@addTagHelper` directives |

### _ViewStart.cshtml

```csharp
@{
    Layout = "_Layout";
}
```

### _ViewImports.cshtml

```csharp
@using OrderManagement
@using OrderManagement.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

> [!summary] Section Summary
> - `Views/` organizes `.cshtml` Razor files by controller name
> - ASP.NET Core uses convention-based view discovery: `Views/{ControllerName}/{ActionName}.cshtml`
> - `_Layout.cshtml` provides the shared page structure; `_ViewStart.cshtml` sets the default layout
> - `_ViewImports.cshtml` declares shared namespaces and tag helpers for all views
