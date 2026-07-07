---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


Tag helpers must be registered in `_ViewImports.cshtml` using the `@addTagHelper` directive:

```cshtml
@* /Views/_ViewImports.cshtml *@

@* Enable all built-in ASP.NET Core tag helpers *@
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

@* Enable custom tag helpers from your application *@
@addTagHelper *, MyApp

@* Enable a specific tag helper only *@
@addTagHelper MyApp.TagHelpers.AlertTagHelper, MyApp
```

### Disabling Tag Helpers

```cshtml
@* Remove a specific tag helper *@
@removeTagHelper Microsoft.AspNetCore.Mvc.TagHelpers.EnvironmentTagHelper, Microsoft.AspNetCore.Mvc.TagHelpers

@* Opt out for a single element using ! prefix *@
<!a asp-controller="Home">This will NOT be processed</!a>
```

### Tag Helper Prefix

To make it explicit which elements are tag helpers (useful in large teams):

```cshtml
@tagHelperPrefix th:

@* Now you must write: *@
<th:a asp-controller="Home" asp-action="Index">Home</th:a>
```

> [!tip] Practical Tip
> Most projects simply use `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` and their own assembly wildcard. The prefix approach is rarely used outside of very large projects with mixed teams.

> [!summary] Section Summary
> - `@addTagHelper` in `_ViewImports.cshtml` enables tag helpers for all views in that directory tree
> - Use wildcard (`*`) to enable all tag helpers from an assembly
> - `@removeTagHelper` disables specific tag helpers; `!` prefix opts out individual elements
> - `@tagHelperPrefix` requires a prefix on all tag helper elements (rarely used)

---
