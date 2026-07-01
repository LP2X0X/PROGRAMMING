---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


Minimal APIs support file uploads through `IFormFile` and `IFormFileCollection` parameter binding.

### Single File Upload

```csharp
app.MapPost("/products/{id}/image", async (
    int id,
    IFormFile file,
    IProductService svc) =>
{
    if (file.Length == 0)
        return Results.BadRequest("File is empty.");

    if (file.Length > 5 * 1024 * 1024) // 5 MB limit
        return Results.BadRequest("File exceeds 5 MB limit.");

    var allowedTypes = new[] { "image/jpeg", "image/png", "image/webp" };
    if (!allowedTypes.Contains(file.ContentType))
        return Results.BadRequest("Only JPEG, PNG, and WebP images are allowed.");

    using var stream = file.OpenReadStream();
    var imageUrl = await svc.UploadImageAsync(id, stream, file.FileName);

    return Results.Ok(new { imageUrl });
})
.DisableAntiforgery()  // Required for API file uploads
.Accepts<IFormFile>("multipart/form-data")
.WithName("UploadProductImage");
```

### Multiple File Upload

```csharp
app.MapPost("/products/{id}/images", async (
    int id,
    IFormFileCollection files,
    IProductService svc) =>
{
    if (files.Count == 0)
        return Results.BadRequest("No files provided.");

    if (files.Count > 5)
        return Results.BadRequest("Maximum 5 files allowed.");

    var urls = new List<string>();
    foreach (var file in files)
    {
        using var stream = file.OpenReadStream();
        var url = await svc.UploadImageAsync(id, stream, file.FileName);
        urls.Add(url);
    }

    return Results.Ok(new { urls });
})
.DisableAntiforgery();
```

### File Upload with Additional Form Fields (.NET 8+)

```csharp
app.MapPost("/products", async (
    [FromForm] string name,
    [FromForm] decimal price,
    [FromForm] IFormFile image,
    IProductService svc) =>
{
    using var stream = image.OpenReadStream();
    var product = await svc.CreateWithImageAsync(name, price, stream, image.FileName);
    return Results.Created($"/api/products/{product.Id}", product);
})
.DisableAntiforgery();
```

Calling the endpoint:

```bash
curl -X POST https://localhost:5001/products \
  -F "name=Widget" \
  -F "price=9.99" \
  -F "image=@photo.jpg"
```

> [!danger] Never Trust the Uploaded Filename
> `IFormFile` exposes a `FileName` property containing the original filename from the client. **Never use this directly** in your code -- attackers can craft filenames to access files they shouldn't (e.g., path traversal attacks like `../../etc/passwd`). Always generate a new name for the file before saving it anywhere.
>
> For more on file upload security threats, see the [Microsoft documentation on file uploads](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads?view=aspnetcore-9.0#security-considerations).
> https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload

> [!warning]
> Always call `.DisableAntiforgery()` on file upload endpoints intended for API consumers. Without it, the endpoint expects an antiforgery token, which API clients do not provide.

> [!summary] Section Summary
> File uploads use `IFormFile` (single) or `IFormFileCollection` (multiple) as handler parameters. Always validate file size and content type. Call `.DisableAntiforgery()` for API-facing upload endpoints. .NET 8+ supports mixed form fields and file uploads using `[FromForm]`.
