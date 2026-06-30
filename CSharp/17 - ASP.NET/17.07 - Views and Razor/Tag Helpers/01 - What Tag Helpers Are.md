---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


**Tag helpers** are C# classes that attach to HTML elements and modify their rendering on the server side. When Razor encounters an HTML element with tag helper attributes (like `asp-controller`), it invokes the corresponding tag helper class, which can:

- Modify the element's attributes (add `href`, `action`, `name`, `id`, etc.)
- Add new child content
- Suppress the element entirely
- Replace the element with different markup

The key insight is that tag helpers **look like HTML**. Compare:

```cshtml
@* HTML Helper (old way) *@
@Html.ActionLink("Products", "Index", "Products", null, new { @class = "nav-link" })

@* Tag Helper (modern way) *@
<a asp-controller="Products" asp-action="Index" class="nav-link">Products</a>
```

The tag helper version is valid HTML that any designer or browser developer tool can parse. The `asp-*` attributes are processed on the server and removed from the final HTML output.

> [!summary] Section Summary
> - Tag helpers are server-side C# classes that enhance HTML elements
> - They use `asp-*` attributes that look like regular HTML attributes
> - They replace the older `@Html.*` helper methods with a more natural, HTML-like syntax
> - The `asp-*` attributes are processed on the server and stripped from the rendered output

---
