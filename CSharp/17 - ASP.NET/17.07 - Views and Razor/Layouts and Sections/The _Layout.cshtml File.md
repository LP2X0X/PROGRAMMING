---
tags:
  - csharp
  - asp-net-core
  - razor
  - layouts
  - views
---


By convention, the main layout file is named `_Layout.cshtml` and lives in `/Views/Shared/`. The underscore prefix indicates it is not a directly routable view.

A minimal layout looks like this:

```cshtml
@* /Views/Shared/_Layout.cshtml *@
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - My Application</title>
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <header>
        <nav>
            <a asp-controller="Home" asp-action="Index">Home</a>
            <a asp-controller="Products" asp-action="Index">Products</a>
            <a asp-controller="Home" asp-action="Contact">Contact</a>
        </nav>
    </header>

    <main class="container">
        @RenderBody()
    </main>

    <footer>
        <p>&copy; @DateTime.Now.Year - My Application</p>
    </footer>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/js/site.js"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

Key elements:
- `@ViewData["Title"]` -- each child view sets this to customize the `<title>` tag
- `@RenderBody()` -- the placeholder where child view content is inserted
- `@await RenderSectionAsync("Scripts", required: false)` -- an optional section for page-specific scripts
- [[Tag Helpers]] like `asp-controller` and `asp-action` generate correct URLs

> [!ad-note] File Location
> The layout does not *have* to be in `/Views/Shared/`. It can be anywhere Razor's view discovery can find it. However, `/Views/Shared/` is the conventional location, and the view engine searches there automatically as a fallback.

> [!summary] Section Summary
> - `_Layout.cshtml` typically lives in `/Views/Shared/`
> - It contains the full HTML document structure with `@RenderBody()` as the content placeholder
> - `ViewData["Title"]` is the standard way to set per-page titles
> - Sections (like `Scripts`) allow child views to inject content into specific layout regions

---
