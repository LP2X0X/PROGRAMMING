---
tags:
  - csharp
  - asp-net-core
  - razor
  - layouts
  - views
---


**Sections** allow child views to inject content into specific, named regions of the layout beyond the main body. The most common use case is page-specific scripts or stylesheets.

### Defining a Section in the Layout

```cshtml
@* In _Layout.cshtml *@
<head>
    <link rel="stylesheet" href="~/css/site.css" />
    @await RenderSectionAsync("Styles", required: false)
</head>
<body>
    <header>...</header>

    <div class="sidebar">
        @await RenderSectionAsync("Sidebar", required: false)
    </div>

    <main>
        @RenderBody()
    </main>

    <script src="~/js/site.js"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
```

### Providing Section Content in a Child View

```cshtml
@model ProductDetailViewModel

<h1>@Model.Name</h1>
<p>@Model.Description</p>

@section Styles {
    <link rel="stylesheet" href="~/css/product-gallery.css" />
}

@section Sidebar {
    <div class="product-filters">
        <h3>Filter by Category</h3>
        @foreach (var cat in Model.Categories)
        {
            <a asp-action="Index" asp-route-category="@cat.Id">@cat.Name</a>
        }
    </div>
}

@section Scripts {
    <script src="~/js/product-gallery.js"></script>
    <script>
        initGallery(@Json.Serialize(Model.Product.ImageUrls));
    </script>
}
```

### Required vs Optional Sections

```cshtml
@* Required: every child view MUST define this section or an error is thrown *@
@await RenderSectionAsync("Title", required: true)

@* Optional: if the child view does not define it, nothing is rendered *@
@await RenderSectionAsync("Scripts", required: false)
```

> [!danger] Runtime Error
> If a section is `required: true` (or `required` is omitted, defaulting to `true` for `RenderSection`) and the child view does not define it, you get an `InvalidOperationException` at runtime. Always set `required: false` for optional sections.

> [!ad-note] Sync vs Async
> Both `RenderSection()` (synchronous) and `RenderSectionAsync()` (asynchronous) exist. Prefer `RenderSectionAsync()` as it handles async tag helpers and view components correctly.

> [!summary] Section Summary
> - Sections let child views inject content into specific layout regions
> - Layouts define section placeholders with `@await RenderSectionAsync("Name", required: false)`
> - Child views provide content with `@section Name { ... }`
> - Sections can be required or optional
> - Common sections: `Scripts`, `Styles`, `Sidebar`, `Breadcrumbs`

---
