---
tags:
  - csharp
  - asp-net-core
  - web-api
  - content-negotiation
  - serialization
---


The `[Produces]` and `[Consumes]` attributes let you ==restrict which media types== a controller or action supports, overriding the global formatter configuration.

### [Produces] — Restricting Response Formats

Apply at the controller or action level to limit what response formats are allowed:

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")] // All actions in this controller return JSON only
public class OrdersController : ControllerBase
{
    [HttpGet]
    public IActionResult GetOrders()
    {
        var orders = _repository.GetAll();
        return Ok(orders); // Always JSON, even if client requests XML
    }

    [HttpGet("{id}/receipt")]
    [Produces("application/pdf")] // Override: this action returns PDF
    public IActionResult GetReceipt(int id)
    {
        var pdf = _receiptService.GeneratePdf(id);
        return File(pdf, "application/pdf");
    }
}
```

Multiple media types:

```csharp
[Produces("application/json", "application/xml")]
public class ProductsController : ControllerBase
{
    // Actions negotiate between JSON and XML only (no CSV even if registered)
}
```

### [Produces] with Response Type

Combine with `typeof` for OpenAPI documentation via [[API Conventions]]:

```csharp
[HttpGet("{id}")]
[Produces("application/json")]
[ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public IActionResult GetProduct(int id)
{
    var product = _repository.GetById(id);
    if (product is null)
        return NotFound();

    return Ok(product);
}
```

### [Consumes] — Restricting Request Formats

Restrict which `Content-Type` values are accepted for request bodies:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    [Consumes("application/json")] // Only accepts JSON request bodies
    public IActionResult CreateProduct([FromBody] CreateProductRequest request)
    {
        var product = _service.Create(request);
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }

    [HttpPost("import")]
    [Consumes("text/csv")] // Only accepts CSV request bodies
    public IActionResult ImportProducts([FromBody] List<ProductDto> products)
    {
        _service.BulkImport(products);
        return Ok(new { imported = products.Count });
    }

    [HttpPost("upload")]
    [Consumes("multipart/form-data")] // Only accepts form data
    public IActionResult UploadImage(IFormFile file)
    {
        // Handle file upload
        return Ok();
    }
}
```

### What Happens When Content-Type Doesn't Match [Consumes]?

If a client sends a request with a `Content-Type` that does not match the `[Consumes]` attribute, ASP.NET Core returns `415 Unsupported Media Type`:

```http
POST /api/products HTTP/1.1
Content-Type: application/xml

<Product><Name>Widget</Name></Product>

--- Response ---
HTTP/1.1 415 Unsupported Media Type
```

> [!example]
> A common pattern is having two endpoints for the same resource — one accepting JSON and another accepting CSV — using `[Consumes]` to route to the correct action:
>
> ```csharp
> [HttpPost]
> [Consumes("application/json")]
> public IActionResult CreateProduct([FromBody] CreateProductRequest request) { ... }
>
> [HttpPost]
> [Consumes("text/csv")]
> public IActionResult CreateProductFromCsv([FromBody] List<ProductDto> products) { ... }
> ```

> [!summary] Section Summary
> `[Produces]` restricts which response formats an action can return. `[Consumes]` restricts which request body formats are accepted. Mismatched `Content-Type` headers trigger a `415 Unsupported Media Type` response. Both attributes are also consumed by OpenAPI/Swagger documentation generators.
