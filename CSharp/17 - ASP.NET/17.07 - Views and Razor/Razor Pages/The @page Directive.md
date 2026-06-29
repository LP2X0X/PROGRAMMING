---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


The **`@page` directive** is what makes a `.cshtml` file a Razor Page. Without it, the file is just a regular Razor view (not routable).

```cshtml
@page
@model IndexModel

<h1>Products</h1>
```

> [!danger] Critical Requirement
> `@page` must be the **first Razor directive** in the file. If `@model` comes before `@page`, the file will not be treated as a Razor Page and will not be routable. You will get a confusing "page not found" error.

`@page` can also include **route template parameters**:

```cshtml
@page "{id:int}"
@* URL: /Products/Edit/42 *@

@page "{id:int?}"
@* URL: /Products/Edit or /Products/Edit/42 (optional parameter) *@

@page "{category}/{id:int}"
@* URL: /Products/electronics/42 *@
```

More on this in the [[#Page Routing and Route Parameters]] section.

> [!summary] Section Summary
> - `@page` is required and must be the first directive -- it makes the file a routable Razor Page
> - Without `@page`, the file is a regular view, not a page
> - `@page` can include route template parameters for dynamic URLs

---
