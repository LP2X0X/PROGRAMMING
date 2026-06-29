---
tags: [csharp, asp-net-core, environments, configuration]
---


Razor Pages and MVC views have a built-in `<environment>` tag helper that conditionally renders HTML based on the current environment.

### Basic Usage

```html
<environment include="Development">
    <!-- Only rendered when ASPNETCORE_ENVIRONMENT is Development -->
    <link rel="stylesheet" href="~/css/site.css" />
    <script src="~/js/site.js"></script>
</environment>

<environment include="Staging,Production">
    <!-- Only rendered in Staging or Production -->
    <link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true" />
    <script src="~/js/site.min.js" asp-append-version="true"></script>
</environment>
```

### Using the exclude Attribute

```html
<environment exclude="Development">
    <!-- Rendered in every environment EXCEPT Development -->
    <link rel="stylesheet"
          href="https://cdn.company.com/css/bootstrap.min.css"
          asp-fallback-href="~/lib/bootstrap/css/bootstrap.min.css"
          asp-fallback-test-class="sr-only"
          asp-fallback-test-property="position"
          asp-fallback-test-value="absolute" />
</environment>
```

### Real-World Example: Debug Toolbar

```html
<environment include="Development">
    <div class="debug-toolbar"
         style="position:fixed; bottom:0; left:0; right:0;
                background:#333; color:#fff; padding:8px; font-size:12px; z-index:9999;">
        <span>Environment: Development</span>
        <span>| Server: @Environment.MachineName</span>
        <span>| Time: @DateTime.Now.ToString("HH:mm:ss")</span>
    </div>
</environment>
```

> [!ad-note] Tag Helper Registration
> The `<environment>` tag helper is available automatically when you include `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` in your `_ViewImports.cshtml`. This is already present in the default project templates.

> [!summary] Section Summary
> - The `<environment>` tag helper conditionally renders HTML blocks based on the current environment.
> - Use `include` to specify which environments should render the content.
> - Use `exclude` to render content in all environments except the listed ones.
> - Common use case: unminified assets in Development, CDN/minified assets in Production.
