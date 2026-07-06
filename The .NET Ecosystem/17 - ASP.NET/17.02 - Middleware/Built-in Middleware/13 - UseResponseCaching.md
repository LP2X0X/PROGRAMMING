---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseResponseCaching

**`UseResponseCaching`** caches HTTP responses on the server based on standard HTTP cache headers (`Cache-Control`, `Vary`, etc.). When a cached response is available, the middleware serves it directly without invoking downstream middleware.

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddResponseCaching();

// Program.cs -- middleware
app.UseResponseCaching();
```

### Using the `[ResponseCache]` Attribute

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // Cache for 60 seconds, vary by Accept-Encoding header
    [HttpGet]
    [ResponseCache(Duration = 60, VaryByHeader = "Accept-Encoding")]
    public IActionResult GetProducts()
    {
        var products = _productService.GetAll();
        return Ok(products);
    }

    // Cache for 120 seconds, vary by category query parameter
    [HttpGet("by-category")]
    [ResponseCache(Duration = 120, VaryByQueryKeys = new[] { "category" })]
    public IActionResult GetByCategory([FromQuery] string category)
    {
        var products = _productService.GetByCategory(category);
        return Ok(products);
    }

    // No caching
    [HttpGet("{id}/inventory")]
    [ResponseCache(NoStore = true, Location = ResponseCacheLocation.None)]
    public IActionResult GetInventoryLevel(int id)
    {
        return Ok(_inventoryService.GetLevel(id));
    }
}
```

### Cache Profile Configuration

```csharp
builder.Services.AddControllersWithViews(options =>
{
    options.CacheProfiles.Add("Default60", new CacheProfile
    {
        Duration = 60,
        Location = ResponseCacheLocation.Any,
        VaryByHeader = "Accept-Encoding"
    });

    options.CacheProfiles.Add("NoCache", new CacheProfile
    {
        NoStore = true,
        Location = ResponseCacheLocation.None
    });
});

// Usage:
[ResponseCache(CacheProfileName = "Default60")]
public IActionResult GetProducts() { }
```

### Key Options

| Option | Description |
|---|---|
| `Duration` | Cache duration in seconds |
| `Location` | `Any`, `Client`, or `None` |
| `VaryByHeader` | Cache separate entries based on header value |
| `VaryByQueryKeys` | Cache separate entries based on query string keys |
| `NoStore` | Prevents caching entirely |

### When You Need It

For endpoints that return data which does not change frequently (product catalogs, reference data, public pages).

### Gotchas

- **Does not cache** when the `Authorization` header is present in the request -- this is by design to prevent serving authenticated content to unauthorized users
- **Does not cache** POST, PUT, DELETE, or PATCH requests -- only GET and HEAD
- The response cache middleware respects `Cache-Control: no-cache` and `no-store` from the client
- This is **server-side** caching within the ASP.NET Core process. It is not a CDN or reverse proxy cache. For high-traffic scenarios, consider a dedicated caching layer
- `VaryByQueryKeys` requires the response caching middleware to be in the pipeline -- it does not work with the `[ResponseCache]` attribute alone when relying on client-side caching

> [!summary] Section Summary
> `UseResponseCaching` caches GET/HEAD responses on the server based on HTTP cache headers. Use `[ResponseCache]` attributes or cache profiles for convenience. It does not cache authenticated responses or non-GET methods. For high-scale applications, supplement with a CDN or distributed cache.
