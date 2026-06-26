---
tags:
  - csharp
  - asp-net-core
  - razor
  - layouts
  - views
---

# Layouts and Sections

> [!ad-note] Overview
> Layouts are Razor's answer to master pages -- they define the common HTML structure (DOCTYPE, head, nav, footer) that wraps every page. Child views plug their content into the layout using `@RenderBody()`, and can inject content into specific layout regions using **sections**. This note covers layout mechanics, sections, nesting, and practical patterns.

## Table of Contents

- [What Layouts Are](#what-layouts-are)
- [The _Layout.cshtml File](#the-_layoutcshtml-file)
- [Setting the Layout](#setting-the-layout)
- [RenderBody -- Inserting Child Content](#renderbody----inserting-child-content)
- [Sections -- Injecting Content into Layout Regions](#sections----injecting-content-into-layout-regions)
- [Conditional Section Content with IsSectionDefined](#conditional-section-content-with-issectiondefined)
- [Multiple Layouts](#multiple-layouts)
- [Nested Layouts](#nested-layouts)
- [No Layout -- Full HTML Control](#no-layout----full-html-control)
- [Real-World Example: E-Commerce Layout](#real-world-example-e-commerce-layout)
- [Comprehensive Summary](#comprehensive-summary)
- [Related Topics](#related-topics)

---

## What Layouts Are

A **layout** is a Razor view that defines the HTML shell shared by multiple pages. Instead of duplicating the `<!DOCTYPE html>`, `<head>`, navigation bar, and footer in every view, you define them once in a layout. Each page view provides only its unique content, which the layout inserts at a designated point.

This is conceptually similar to:
- Template inheritance in Django/Jinja2
- Master pages in ASP.NET Web Forms
- Slots in Vue.js or Svelte

The benefits are straightforward:
- **DRY principle**: common markup lives in one place
- **Consistency**: every page shares the same navigation, footer, and asset references
- **Maintainability**: updating the nav or adding a new CSS file is a single change

> [!summary] Section Summary
> - Layouts define the shared HTML structure (head, nav, footer) for an application
> - Child views provide only their unique content
> - This eliminates duplication and ensures visual consistency across pages

---

## The _Layout.cshtml File

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

## Setting the Layout

There are three ways to specify which layout a view uses, in order of precedence:

### 1. Per-View (Highest Priority)

Set `Layout` directly in the view's code block:

```cshtml
@{
    Layout = "_SpecialLayout";
}

<h1>This page uses a special layout</h1>
```

### 2. Via _ViewStart.cshtml (Default for All Views)

The most common approach. `_ViewStart.cshtml` runs before every view and sets the default layout:

```cshtml
@* /Views/_ViewStart.cshtml *@
@{
    Layout = "_Layout";
}
```

### 3. Conditional Layout in _ViewStart.cshtml

```cshtml
@* /Views/_ViewStart.cshtml *@
@{
    if (Context.Request.Path.StartsWithSegments("/admin"))
    {
        Layout = "_AdminLayout";
    }
    else
    {
        Layout = "_Layout";
    }
}
```

> [!tip] Practical Tip
> For area-based layout switching, place a separate `_ViewStart.cshtml` in the area's Views folder rather than using conditionals in the root `_ViewStart.cshtml`. This is cleaner and scales better.
>
> ```
> /Views/_ViewStart.cshtml         → Layout = "_Layout"
> /Areas/Admin/Views/_ViewStart.cshtml → Layout = "_AdminLayout"
> ```

**Layout name resolution:**
- If you specify `"_Layout"`, Razor searches:
  1. The same folder as the current view
  2. `/Views/Shared/`
  3. `/Pages/Shared/` (for Razor Pages)
- You can also specify a full path: `"/Views/Shared/_SpecialLayout.cshtml"`

> [!summary] Section Summary
> - Layout can be set per-view, in `_ViewStart.cshtml`, or conditionally
> - Per-view `Layout` assignment overrides `_ViewStart.cshtml`
> - Layout names are resolved by searching the view's folder, then `/Views/Shared/`
> - Area-specific `_ViewStart.cshtml` files provide clean layout switching

---

## RenderBody -- Inserting Child Content

`@RenderBody()` is the single most important method in a layout. It marks where the child view's content is inserted. **Every layout must call `@RenderBody()` exactly once.**

```cshtml
@* In _Layout.cshtml *@
<main class="container">
    @RenderBody()
</main>
```

When a child view like `/Views/Products/Details.cshtml` renders:

```cshtml
@model ProductViewModel

<h1>@Model.Name</h1>
<p>@Model.Description</p>
<p>Price: @Model.Price.ToString("C")</p>
```

The final HTML output combines the layout's structure with the child view's content:

```html
<!DOCTYPE html>
<html>
<head>...</head>
<body>
    <header>...</header>
    <main class="container">
        <h1>Widget Pro</h1>
        <p>The finest widget money can buy.</p>
        <p>Price: $29.99</p>
    </main>
    <footer>...</footer>
</body>
</html>
```

> [!warning] Common Misconception
> `@RenderBody()` does not accept parameters. You cannot pass data to it or specify which view to render. It always renders the content of the current child view. If you need multiple replaceable content areas, use **sections**.

> [!summary] Section Summary
> - `@RenderBody()` inserts the child view's entire content at that point in the layout
> - It must be called exactly once per layout
> - It takes no parameters and always renders the current child view
> - For multiple insertion points, use sections

---

## Sections -- Injecting Content into Layout Regions

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

## Conditional Section Content with IsSectionDefined

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

## Multiple Layouts

Real applications often need different layouts for different areas. Common examples:
- Public site layout (marketing header, footer)
- Admin dashboard layout (sidebar navigation, minimal footer)
- Authentication layout (centered card, no navigation)
- Email/print layout (no interactive elements)

```
/Views/Shared/
    _Layout.cshtml           ← public site
    _AdminLayout.cshtml      ← admin dashboard
    _AuthLayout.cshtml       ← login/register pages
    _PrintLayout.cshtml      ← print-friendly pages
```

### Switching Layouts per View

```cshtml
@* /Views/Account/Login.cshtml *@
@{
    Layout = "_AuthLayout";
}

<div class="login-card">
    <h1>Sign In</h1>
    @* login form *@
</div>
```

### Switching Layouts per Area

```cshtml
@* /Areas/Admin/Views/_ViewStart.cshtml *@
@{
    Layout = "_AdminLayout";
}
```

### Switching Layouts per Controller

```cshtml
@* /Views/_ViewStart.cshtml *@
@{
    var controller = Context.Request.RouteValues["controller"]?.ToString();
    Layout = controller == "Account" ? "_AuthLayout" : "_Layout";
}
```

> [!tip] Practical Tip
> Keep layout switching simple. Prefer separate `_ViewStart.cshtml` files in area/folder hierarchies over complex conditional logic. If your `_ViewStart.cshtml` has more than a simple `if/else`, consider reorganizing your view folder structure.

> [!summary] Section Summary
> - Applications commonly have multiple layouts for different areas (public, admin, auth)
> - Layouts can be switched per-view, per-area (via folder-level `_ViewStart.cshtml`), or conditionally
> - Prefer folder-level `_ViewStart.cshtml` files over complex conditionals for cleaner organization

---

## Nested Layouts

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

## No Layout -- Full HTML Control

Some views need to render a complete HTML document without any layout wrapping. Set `Layout = null`:

```cshtml
@{
    Layout = null;
}

<!DOCTYPE html>
<html>
<head>
    <title>Error</title>
    <style>
        body { font-family: sans-serif; text-align: center; padding: 50px; }
    </style>
</head>
<body>
    <h1>Something went wrong</h1>
    <p>Please try again later.</p>
</body>
</html>
```

Common scenarios for `Layout = null`:
- **Error pages**: the layout itself might be broken, so the error page must be self-contained
- **Login/auth pages**: when they need a completely different HTML structure
- **Email templates**: rendered to HTML strings, not served as web pages
- **Embedded widgets**: HTML fragments served inside iframes
- **PDF generation**: HTML templates that a PDF renderer processes

> [!summary] Section Summary
> - `Layout = null` renders the view as a standalone HTML document with no layout wrapper
> - Appropriate for error pages, email templates, embedded widgets, and PDF generation
> - The view must include the complete HTML structure (`<!DOCTYPE>`, `<html>`, `<head>`, `<body>`)

---

## Real-World Example: E-Commerce Layout

A complete `_Layout.cshtml` for a typical e-commerce site:

```cshtml
@* /Views/Shared/_Layout.cshtml *@
@inject Microsoft.AspNetCore.Hosting.IWebHostEnvironment Env
@inject IStringLocalizer<SharedResource> Localizer

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - ShopMaster</title>
    <meta name="description"
          content="@(ViewData["MetaDescription"] ?? "ShopMaster - Your favorite online store")" />

    <link rel="stylesheet"
          href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
    @await RenderSectionAsync("Styles", required: false)
</head>
<body>
    @* Top announcement bar *@
    @if (IsSectionDefined("AnnouncementBar"))
    {
        @await RenderSectionAsync("AnnouncementBar")
    }

    @* Navigation *@
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" asp-controller="Home" asp-action="Index">
                ShopMaster
            </a>
            <button class="navbar-toggler" type="button"
                    data-bs-toggle="collapse" data-bs-target="#mainNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="mainNav">
                <ul class="navbar-nav me-auto">
                    <li class="nav-item">
                        <a class="nav-link"
                           asp-controller="Products" asp-action="Index">
                            @Localizer["Products"]
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link"
                           asp-controller="Categories" asp-action="Index">
                            @Localizer["Categories"]
                        </a>
                    </li>
                </ul>
                <ul class="navbar-nav">
                    <li class="nav-item">
                        <a class="nav-link" asp-controller="Cart" asp-action="Index">
                            Cart
                            <vc:cart-count />
                        </a>
                    </li>
                    <partial name="_LoginPartial" />
                </ul>
            </div>
        </div>
    </nav>

    @* Breadcrumbs *@
    @if (IsSectionDefined("Breadcrumbs"))
    {
        <div class="container mt-2">
            @await RenderSectionAsync("Breadcrumbs")
        </div>
    }

    @* Main content *@
    <main class="container py-4">
        @RenderBody()
    </main>

    @* Footer *@
    <footer class="bg-light py-4 mt-5">
        <div class="container">
            <div class="row">
                <div class="col-md-4">
                    <h5>ShopMaster</h5>
                    <p>&copy; @DateTime.Now.Year ShopMaster Inc.</p>
                </div>
                <div class="col-md-4">
                    <h5>Quick Links</h5>
                    <ul class="list-unstyled">
                        <li><a asp-controller="Home" asp-action="About">About</a></li>
                        <li><a asp-controller="Home" asp-action="Contact">Contact</a></li>
                        <li><a asp-controller="Home" asp-action="Privacy">Privacy</a></li>
                    </ul>
                </div>
                <div class="col-md-4">
                    <h5>@Localizer["CustomerService"]</h5>
                    <p>support@@shopmaster.com</p>
                    <p>1-800-SHOP-NOW</p>
                </div>
            </div>
        </div>
    </footer>

    @* Scripts *@
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js">
    </script>
    <script src="~/js/site.js" asp-append-version="true"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

This layout demonstrates:
- `@RenderBody()` for main content
- Multiple optional sections (`Styles`, `Scripts`, `AnnouncementBar`, `Breadcrumbs`)
- `IsSectionDefined()` for conditional section rendering
- `@inject` for localization and environment info
- [[Tag Helpers]] for URL generation and cache-busting (`asp-append-version`)
- [[Partial Views and View Components|Partial views and view components]] (`_LoginPartial`, `<vc:cart-count />`)
- `@@` for escaping the `@` in the email address
- `ViewData["Title"]` for per-page titles

> [!summary] Section Summary
> - A production layout combines `@RenderBody()`, multiple sections, and shared components
> - `IsSectionDefined()` enables smart defaults for optional regions like breadcrumbs
> - Tag helpers, partial views, view components, and DI all work seamlessly in layouts
> - `ViewData` is the standard mechanism for passing simple values (title, meta description) from child views to the layout

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Layouts** in ASP.NET Core Razor provide a master page mechanism where shared HTML structure (DOCTYPE, head, nav, footer) is defined once and reused by all views. The layout file (`_Layout.cshtml`, typically in `/Views/Shared/`) contains `@RenderBody()` as the insertion point for child view content.
>
> **Setting the layout** happens in three places: `_ViewStart.cshtml` (default for all views), per-view assignment (`Layout = "_SpecialLayout"`), or conditionally based on route/area. Per-view settings override `_ViewStart.cshtml`. For area-based switching, prefer separate `_ViewStart.cshtml` files in each area's Views folder.
>
> **Sections** (`@section Name { }` in child views, `@await RenderSectionAsync("Name", required: false)` in layouts) allow child views to inject content into specific layout regions beyond the main body. Common sections include `Scripts`, `Styles`, and `Breadcrumbs`. `IsSectionDefined()` enables fallback content when a section is not provided.
>
> **Multiple layouts** serve different application areas (public site, admin, auth). **Nested layouts** create hierarchies where a child layout uses a parent layout, but sections must be explicitly forwarded between levels. **`Layout = null`** gives a view full HTML control with no layout wrapping, useful for error pages and email templates.
>
> All of these mechanisms integrate naturally with [[Razor Syntax]], [[Tag Helpers]], [[Partial Views and View Components]], and the broader ASP.NET Core MVC pipeline.

---

## Related Topics

- [[Razor Syntax]] -- the foundational `@` syntax used within layouts
- [[Partial Views and View Components]] -- reusable fragments rendered within layouts and views
- [[Tag Helpers]] -- `asp-controller`, `asp-action`, `asp-append-version` used in layout navigation
- [[Razor Pages]] -- Razor Pages use the same layout system as MVC views
- [[17.06 - Controllers and Actions]] -- controllers set `ViewData` that layouts consume
- [[17.02 - Middleware]] -- the request pipeline that ultimately renders layouts
