---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


There are two ways to invoke a view component from a Razor view.

### Tag Helper Syntax (Recommended)

```cshtml
@* Simple invocation *@
<vc:cart-summary />

@* With parameters *@
<vc:recent-products max-items="5" />

@* With a string parameter *@
<vc:navigation-menu area="admin" />
```

The tag helper uses **kebab-case**: `CartSummary` becomes `<vc:cart-summary>`, and `maxItems` becomes `max-items`.

> [!ad-note] Enabling the vc: Tag Helper
> The `<vc:>` tag helper requires `@addTagHelper` in `_ViewImports.cshtml`:
> ```cshtml
> @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
> ```
> This is the same directive that enables built-in [[Tag Helpers]] and is almost always already present.

### Component.InvokeAsync Syntax

```cshtml
@await Component.InvokeAsync("CartSummary")

@await Component.InvokeAsync("RecentProducts", new { maxItems = 5 })
```

This uses the component name as a string, which means no compile-time checking. Prefer the tag helper approach.

> [!tip] Practical Tip
> Use `<vc:component-name />` in views for readability and compile-time safety. Use `Component.InvokeAsync()` only when you need to invoke a view component dynamically (e.g., the component name comes from configuration).

> [!summary] Section Summary
> - `<vc:component-name />` is the recommended tag helper syntax (kebab-case)
> - `Component.InvokeAsync("Name")` is the programmatic alternative (string-based)
> - Parameters map from PascalCase to kebab-case: `maxItems` becomes `max-items`
> - The `vc:` prefix requires `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers`

---
