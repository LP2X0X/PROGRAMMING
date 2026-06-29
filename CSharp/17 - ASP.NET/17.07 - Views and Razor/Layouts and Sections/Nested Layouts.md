---
tags:
  - csharp
  - asp-net-core
  - razor
  - layouts
  - views
---


A layout can itself use another layout, creating a hierarchy. This is useful when you have a base layout with the HTML skeleton, and specialized layouts that add area-specific structure.

```cshtml
@* /Views/Shared/_BaseLayout.cshtml -- the outermost shell *@
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <title>@ViewData["Title"] - My App</title>
    <link rel="stylesheet" href="~/css/site.css" />
    @await RenderSectionAsync("Styles", required: false)
</head>
<body>
    @RenderBody()

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

```cshtml
@* /Views/Shared/_AdminLayout.cshtml -- uses _BaseLayout *@
@{
    Layout = "_BaseLayout";
}

<div class="admin-wrapper">
    <aside class="admin-sidebar">
        <nav>
            <a asp-area="Admin" asp-controller="Dashboard" asp-action="Index">Dashboard</a>
            <a asp-area="Admin" asp-controller="Users" asp-action="Index">Users</a>
            <a asp-area="Admin" asp-controller="Products" asp-action="Index">Products</a>
        </nav>
    </aside>
    <main class="admin-content">
        @RenderBody()
    </main>
</div>

@section Styles {
    <link rel="stylesheet" href="~/css/admin.css" />
}
```

The rendering chain: **Child View** -> **_AdminLayout** -> **_BaseLayout**

> [!warning] Common Misconception
> Sections do NOT pass through nested layouts automatically. If the child view defines `@section Scripts { ... }`, the intermediate layout (`_AdminLayout`) must explicitly render or re-declare it for the outermost layout to see it. Each layout level only sees sections defined by its direct child.

To pass sections through nested layouts:

```cshtml
@* In _AdminLayout.cshtml -- forwarding the Scripts section to _BaseLayout *@
@if (IsSectionDefined("Scripts"))
{
    @section Scripts {
        @await RenderSectionAsync("Scripts")
    }
}
```

> [!summary] Section Summary
> - Layouts can reference other layouts, creating a hierarchy
> - The outermost layout contains the HTML skeleton; inner layouts add structural regions
> - Sections do NOT automatically pass through nested layouts -- they must be explicitly forwarded
> - Common pattern: `_BaseLayout` (HTML shell) -> `_AdminLayout` (sidebar + content area) -> child view

---
