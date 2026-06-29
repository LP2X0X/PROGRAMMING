---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseStaticFiles

**`UseStaticFiles`** serves static files (HTML, CSS, JavaScript, images, etc.) directly from the `wwwroot` directory without passing through the full middleware pipeline. It acts as a **short-circuit** middleware -- if a matching file is found, it serves it immediately and does not call the next middleware.

### Basic Configuration

```csharp
// Serves files from wwwroot/
app.UseStaticFiles();
```

### Serving from a Custom Directory

```csharp
// Serve files from a custom directory at a custom URL path
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "ProductImages")),
    RequestPath = "/images"
});
```

This maps requests like `/images/product-001.jpg` to the file `ProductImages/product-001.jpg` on disk.

### Setting Cache Headers

```csharp
app.UseStaticFiles(new StaticFileOptions
{
    OnPrepareResponse = ctx =>
    {
        // Cache static files for 30 days
        ctx.Context.Response.Headers.Append(
            "Cache-Control", "public,max-age=2592000");
    }
});
```

### Serving Files with Custom Content Types

```csharp
var contentTypeProvider = new FileExtensionContentTypeProvider();
contentTypeProvider.Mappings[".webmanifest"] = "application/manifest+json";
contentTypeProvider.Mappings[".data"] = "application/octet-stream";

app.UseStaticFiles(new StaticFileOptions
{
    ContentTypeProvider = contentTypeProvider
});
```

### When You Need It

Any application that serves static assets (CSS, JS, images, fonts). Placed **before** `UseRouting` so static file requests are handled quickly without routing overhead.

### Gotchas

- By default, only files in `wwwroot` are served. Files outside this directory are not accessible unless you explicitly configure a `PhysicalFileProvider`
- `UseStaticFiles` does **not** enable directory browsing by default. Use `UseDirectoryBrowser` separately if needed (not recommended in production)
- Files without a recognized MIME type are not served. Use `ServeUnknownFileTypes = true` with caution -- it can expose unintended files
- Place `UseStaticFiles` **before** `UseRouting` and `UseAuthorization` so that static file requests do not incur authentication/authorization overhead

> [!summary] Section Summary
> `UseStaticFiles` serves files from `wwwroot` (or custom paths) as a short-circuit middleware. Configure cache headers for performance, use `PhysicalFileProvider` for custom directories, and always place it before routing to avoid unnecessary processing.
