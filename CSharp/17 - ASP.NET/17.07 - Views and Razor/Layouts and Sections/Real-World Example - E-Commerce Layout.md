---
tags:
  - csharp
  - asp-net-core
  - razor
  - layouts
  - views
---


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
