---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


Two special files control shared behavior across all views in a folder (or the entire application).

### _ViewImports.cshtml

`_ViewImports.cshtml` contains directives that are automatically applied to every view in the same directory and all subdirectories:

```cshtml
@* /Views/_ViewImports.cshtml *@
@using MyApp
@using MyApp.Models
@using MyApp.ViewModels
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper MyApp.TagHelpers.*, MyApp
```

**Key behaviors:**
- Directives cascade downward through subdirectories
- You can place `_ViewImports.cshtml` at multiple levels -- they are additive
- Supported directives: `@using`, `@addTagHelper`, `@removeTagHelper`, `@tagHelperPrefix`, `@model`, `@inject`

A common pattern is a root-level `_ViewImports.cshtml` for application-wide imports, and folder-level ones for area-specific imports.

### _ViewStart.cshtml

`_ViewStart.cshtml` contains C# code that runs before every view renders. Its primary use is setting the default [[Layouts and Sections|layout]]:

```cshtml
@* /Views/_ViewStart.cshtml *@
@{
    Layout = "_Layout";
}
```

**Key behaviors:**
- Runs before the view's own code
- Can contain conditional logic (e.g., different layouts per area)
- Cascades downward like `_ViewImports.cshtml`
- Can be placed at multiple directory levels

```cshtml
@* /Views/Admin/_ViewStart.cshtml *@
@{
    Layout = "_AdminLayout";
}
```

> [!ad-note] Execution Order
> When a view renders: `_ViewStart.cshtml` runs first (setting layout and other defaults), then the view itself executes, then the layout wraps the result. If there are nested `_ViewStart.cshtml` files, they run from outermost to innermost.

> [!summary] Section Summary
> - `_ViewImports.cshtml` provides shared `@using`, `@addTagHelper`, and other directives to all views in its directory tree
> - `_ViewStart.cshtml` runs code before every view (typically sets the default layout)
> - Both cascade downward through subdirectories and can exist at multiple levels
> - The underscore prefix is a convention indicating these are not directly renderable views

---
