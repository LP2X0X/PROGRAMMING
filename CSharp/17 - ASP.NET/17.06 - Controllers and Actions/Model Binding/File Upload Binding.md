---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


ASP.NET Core provides built-in support for binding uploaded files through the `IFormFile` and `IFormFileCollection` interfaces.

### Single File Upload

```csharp
[HttpPost("products/{id}/image")]
public async Task<IActionResult> UploadImage(
    [FromRoute] int id,
    IFormFile image)
{
    if (image == null || image.Length == 0)
        return BadRequest("No file uploaded.");
    
    // Available properties:
    // image.FileName       -> original file name from the client
    // image.Length          -> file size in bytes
    // image.ContentType    -> MIME type (e.g., "image/png")
    // image.Name           -> the form field name
    
    // Size validation
    const long maxSize = 5 * 1024 * 1024; // 5 MB
    if (image.Length > maxSize)
        return BadRequest($"File exceeds maximum size of {maxSize / 1024 / 1024} MB.");
    
    // Content type validation
    var allowedTypes = new[] { "image/jpeg", "image/png", "image/webp" };
    if (!allowedTypes.Contains(image.ContentType))
        return BadRequest("Only JPEG, PNG, and WebP images are allowed.");
    
    // Save to disk
    var filePath = Path.Combine("uploads", $"{id}_{Guid.NewGuid()}{Path.GetExtension(image.FileName)}");
    
    using (var stream = new FileStream(filePath, FileMode.Create))
    {
        await image.CopyToAsync(stream);
    }
    
    return Ok(new { path = filePath });
}
```

### Multiple File Uploads

```csharp
[HttpPost("products/{id}/photos")]
public async Task<IActionResult> UploadPhotos(
    [FromRoute] int id,
    IFormFileCollection photos)  // or List<IFormFile> photos
{
    if (photos.Count == 0)
        return BadRequest("No files uploaded.");
    
    var uploadedPaths = new List<string>();
    
    foreach (var photo in photos)
    {
        if (photo.Length > 0)
        {
            var filePath = Path.Combine("uploads", $"{id}_{Guid.NewGuid()}{Path.GetExtension(photo.FileName)}");
            using var stream = new FileStream(filePath, FileMode.Create);
            await photo.CopyToAsync(stream);
            uploadedPaths.Add(filePath);
        }
    }
    
    return Ok(new { count = uploadedPaths.Count, paths = uploadedPaths });
}
```

### The HTML Form

The form **must** use `enctype="multipart/form-data"` for file uploads to work:

```html
<form method="post"
      action="/products/5/photos"
      enctype="multipart/form-data">
    
    <input type="file" name="photos" multiple />
    <button type="submit">Upload Photos</button>
</form>
```

```ad-danger
Never trust the file name or content type from the client. Always validate and sanitize:
- Generate a new file name server-side (do not use the client-provided name directly for storage)
- Validate the content type and file extension
- Enforce size limits
- Consider scanning for malware in production systems
- Do not save files to a location within the web root where they could be executed
```
