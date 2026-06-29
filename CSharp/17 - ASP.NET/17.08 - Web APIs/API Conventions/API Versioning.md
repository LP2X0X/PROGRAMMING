---
tags:
  - csharp
  - asp-net-core
  - web-api
  - conventions
  - openapi
---


**API versioning** allows you to evolve your API without breaking existing consumers. ==When you need to introduce breaking changes, a new version lets old clients continue using the previous contract while new clients adopt the updated one.==

### Installing the Versioning Package

```bash
dotnet add package Asp.Versioning.Mvc
dotnet add package Asp.Versioning.Mvc.ApiExplorer
```

### Configuring API Versioning

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true; // Adds api-supported-versions header
    options.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),
        new QueryStringApiVersionReader("api-version"),
        new HeaderApiVersionReader("X-Api-Version"),
        new MediaTypeApiVersionReader("v")
    );
})
.AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});
```

### Versioning Strategies

#### URL Segment Versioning (Most Common)

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0")]
public class ProductsV1Controller : ControllerBase
{
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDtoV1>> GetProduct(int id)
    {
        // V1 response shape
        return new ProductDtoV1
        {
            Id = id,
            Name = "Wireless Mouse",
            Price = 29.99m
        };
    }
}

[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("2.0")]
public class ProductsV2Controller : ControllerBase
{
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDtoV2>> GetProduct(int id)
    {
        // V2 response includes additional fields
        return new ProductDtoV2
        {
            Id = id,
            Name = "Wireless Mouse",
            Price = new MoneyDto { Amount = 29.99m, Currency = "USD" },
            Sku = "WM-001",
            CreatedAt = DateTime.UtcNow
        };
    }
}
```

Requests:

```http
GET /api/v1/products/42 HTTP/1.1
Host: localhost:5001
```

```http
GET /api/v2/products/42 HTTP/1.1
Host: localhost:5001
```

#### Query String Versioning

```http
GET /api/products/42?api-version=2.0 HTTP/1.1
Host: localhost:5001
```

#### Header Versioning

```http
GET /api/products/42 HTTP/1.1
Host: localhost:5001
X-Api-Version: 2.0
```

### Deprecating API Versions

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0", Deprecated = true)] // Marks v1 as deprecated
[ApiVersion("2.0")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    [MapToApiVersion("1.0")]
    public ActionResult<ProductDtoV1> GetProductV1(int id) { /* ... */ }

    [HttpGet("{id}")]
    [MapToApiVersion("2.0")]
    public ActionResult<ProductDtoV2> GetProductV2(int id) { /* ... */ }
}
```

Deprecated versions include an `api-deprecated-versions` response header, signaling to consumers that they should migrate.

### Swagger Integration with Versioning

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Product API",
        Version = "v1",
        Description = "Version 1.0 - Deprecated"
    });
    options.SwaggerDoc("v2", new OpenApiInfo
    {
        Title = "Product API",
        Version = "v2",
        Description = "Version 2.0 - Current"
    });
});

// In the middleware pipeline
app.UseSwaggerUI(options =>
{
    options.SwaggerEndpoint("/swagger/v1/swagger.json", "Product API v1 (Deprecated)");
    options.SwaggerEndpoint("/swagger/v2/swagger.json", "Product API v2");
});
```

> [!tip]
> URL segment versioning (`/api/v2/...`) is the most widely adopted strategy because it is the most visible, cacheable, and easy to route. Query string and header versioning can be useful for internal APIs or when URL changes are undesirable.

> [!summary] Section Summary
> API versioning uses `Asp.Versioning.Mvc` to support URL segment, query string, header, and media type strategies. URL segment versioning is the most common. Deprecated versions are signaled via response headers. Swagger can be configured to show separate docs per version.
