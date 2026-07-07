---
tags: [csharp, asp-net-core, project-structure]
---


The `wwwroot/` folder is the web root directory. Any file placed here is directly accessible via HTTP without any controller or page handling the request.

```
wwwroot/
  css/
    site.css
  js/
    site.js
  lib/
    bootstrap/
    jquery/
    jquery-validation/
    jquery-validation-unobtrusive/
  images/
    logo.png
  favicon.ico
```

### Serving Static Files

Static file serving is enabled by the `UseStaticFiles()` middleware in `Program.cs`:

```csharp
app.UseStaticFiles();
```

A file at `wwwroot/css/site.css` is accessible at `https://localhost:5001/css/site.css`. Notice the URL path does not include `wwwroot` -- the web root is implicit.

> [!example] Referencing Static Files in Razor Views
> ```html
> <link rel="stylesheet" href="~/css/site.css" />
> <script src="~/js/site.js"></script>
> <img src="~/images/logo.png" alt="Company Logo" />
> ```
> The `~` (tilde) resolves to the web root path. This is a Razor convention, not standard HTML.

> [!warning] Security Consideration
> Only files inside `wwwroot/` are served to clients. Files outside this folder (such as `.cs` source files, `appsettings.json`, etc.) are never exposed. Never place sensitive files inside `wwwroot/`.

### Web API Projects

Web API projects typically do not include a `wwwroot/` folder since they serve JSON data rather than HTML pages. However, you can add one manually if your API needs to serve static assets (e.g., API documentation files, images).

> [!summary] Section Summary
> - `wwwroot/` is the web root; all files inside are publicly accessible via HTTP
> - `UseStaticFiles()` middleware must be added to enable static file serving
> - URLs do not include `wwwroot` in the path -- it is the implicit root
> - Only files inside `wwwroot/` are served; all other project files are protected
