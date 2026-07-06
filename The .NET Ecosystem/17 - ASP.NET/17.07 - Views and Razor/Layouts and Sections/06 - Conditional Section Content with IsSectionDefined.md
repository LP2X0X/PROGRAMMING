---
tags:
  - csharp
  - asp-net-core
  - razor
  - layouts
  - views
---


`IsSectionDefined()` lets the layout provide **default content** when a child view does not define a section:

```cshtml
@* In _Layout.cshtml *@
<div class="sidebar">
    @if (IsSectionDefined("Sidebar"))
    {
        @await RenderSectionAsync("Sidebar")
    }
    else
    {
        <partial name="_DefaultSidebar" />
    }
</div>
```

This pattern is useful for:
- Default sidebar content that most pages share, but some pages override
- Fallback meta descriptions or Open Graph tags
- Default breadcrumb navigation

```cshtml
@* Another example: conditional page-specific styles with a default *@
<head>
    <link rel="stylesheet" href="~/css/site.css" />
    @if (IsSectionDefined("Styles"))
    {
        @await RenderSectionAsync("Styles")
    }
</head>
```

> [!summary] Section Summary
> - `IsSectionDefined("Name")` checks whether the child view defined a particular section
> - Enables fallback/default content patterns in the layout
> - Useful for sidebars, meta tags, and breadcrumbs where most pages share a default

---
