---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


| Aspect | Tag Helpers | HTML Helpers |
|---|---|---|
| **Syntax** | HTML-like (`<a asp-action>`) | C# method calls (`@Html.ActionLink()`) |
| **Readability** | Looks like standard HTML | Breaks the HTML flow |
| **Designer-friendly** | Yes -- valid HTML attributes | No -- requires C# knowledge |
| **IntelliSense** | Full support in Visual Studio | Full support |
| **Custom creation** | Inherit from `TagHelper` | Write extension methods on `IHtmlHelper` |
| **Nesting** | Natural HTML nesting | Clunky lambda expressions |
| **CSS class** | `class="..."` (normal HTML) | `new { @class = "..." }` (anonymous object) |

```cshtml
@* HTML Helper *@
@Html.ActionLink("Products", "Index", "Products", null, new { @class = "nav-link" })

@* Equivalent Tag Helper *@
<a asp-controller="Products" asp-action="Index" class="nav-link">Products</a>
```

```cshtml
@* HTML Helper for a form field *@
@Html.TextBoxFor(m => m.Name, new { @class = "form-control", placeholder = "Enter name" })

@* Equivalent Tag Helper *@
<input asp-for="Name" class="form-control" placeholder="Enter name" />
```

> [!ad-note] Migration Note
> HTML helpers still work in ASP.NET Core and are not deprecated. However, tag helpers are the recommended approach for new development. You can mix both in the same view during a gradual migration.

> [!summary] Section Summary
> - Tag helpers provide an HTML-native syntax that is more readable and designer-friendly
> - HTML helpers use C# method calls that break the HTML flow
> - Tag helpers are the recommended approach for new projects
> - Both coexist in the same view -- no need for a full migration

---
