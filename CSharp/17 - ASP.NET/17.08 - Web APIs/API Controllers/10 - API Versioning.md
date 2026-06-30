---
tags:
  - csharp
  - asp-net-core
  - web-api
  - controllers
---


As your API evolves, you need to introduce breaking changes without breaking existing clients. **API versioning** allows multiple versions to coexist.

### Installing the Versioning Package

```bash
dotnet add package Asp.Versioning.Mvc
dotnet add package Asp.Versioning.Mvc.ApiExplorer
```

### Configuration in Program.cs

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;  // Adds api-supported-versions header
    
    // Choose one or combine multiple readers
    options.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),
        new QueryStringApiVersionReader("api-version"),
        new HeaderApiVersionReader("X-Api-Version")
    );
})
.AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});
```

### Approach 1: URL Path Versioning

==The most common and recommended approach.== The version is part of the URL path.

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0")]
public class ProductsV1Controller : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<ProductV1Dto>> GetAll()
    {
        // V1 implementation
    }
}

[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("2.0")]
public class ProductsV2Controller : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<ProductV2Dto>> GetAll()
    {
        // V2 implementation with additional fields
    }
}
```

```http
GET /api/v1/products
GET /api/v2/products
```

### Approach 2: Query String Versioning

```http
GET /api/products?api-version=1.0
GET /api/products?api-version=2.0
```

### Approach 3: Header Versioning

```http
GET /api/products
X-Api-Version: 1.0
```

### Deprecating API Versions

Mark old versions as deprecated to signal clients:

```csharp
[ApiVersion("1.0", Deprecated = true)]  // Still works, but marked deprecated
[ApiVersion("2.0")]
[ApiController]
[Route("api/v{version:apiVersion}/products")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public ActionResult<IEnumerable<ProductV1Dto>> GetAllV1()
    {
        // Legacy implementation
    }

    [HttpGet]
    [MapToApiVersion("2.0")]
    public ActionResult<IEnumerable<ProductV2Dto>> GetAllV2()
    {
        // Current implementation
    }
}
```

The response includes deprecation headers:

```http
api-supported-versions: 1.0, 2.0
api-deprecated-versions: 1.0
```

### Comparison of Approaches

| Approach | Pros | Cons |
|---|---|---|
| URL path (`/v1/`) | Explicit, easy to test, cacheable | URL changes per version |
| Query string (`?api-version=1`) | URL stays the same | Easy to forget, not RESTful |
| Header (`X-Api-Version`) | Clean URLs | Hidden, harder to test in browser |

> [!tip]
> URL path versioning is the most widely used approach in the industry. APIs like GitHub, Stripe, and Twilio all use URL path versioning. Start with this unless you have a specific reason not to.

> [!summary] Section Summary
> API versioning lets you evolve your API without breaking existing clients. URL path versioning (`/api/v1/`, `/api/v2/`) is the most common and recommended approach. Use the `Asp.Versioning.Mvc` package to configure versioning, and mark old versions as deprecated to guide clients toward upgrading.
