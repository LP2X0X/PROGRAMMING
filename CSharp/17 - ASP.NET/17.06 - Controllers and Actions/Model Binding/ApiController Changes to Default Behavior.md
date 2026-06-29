---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


The `[ApiController]` attribute (applied to controllers inheriting from `ControllerBase`) changes the default binding source inference rules significantly.

### Default Inference Rules with [ApiController]

| Parameter Type | Inferred Source |
|---|---|
| Complex types (classes, records) | `[FromBody]` (JSON) |
| Simple types matching a route parameter | `[FromRoute]` |
| Simple types not matching a route parameter | `[FromQuery]` |
| `IFormFile`, `IFormFileCollection` | `[FromForm]` |
| `CancellationToken` | `HttpContext.RequestAborted` |

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // Complex type -> automatically [FromBody]
    [HttpPost]
    public IActionResult Create(CreateProductRequest request)
    {
        // No [FromBody] needed -- [ApiController] infers it
        // Expects JSON body
    }
    
    // Simple type matching route param -> [FromRoute]
    // Simple type not in route -> [FromQuery]
    [HttpGet("{id}")]
    public IActionResult Get(int id, bool includeReviews)
    {
        // id -> [FromRoute] (matches {id} in route template)
        // includeReviews -> [FromQuery] (no matching route param)
        // GET /api/products/5?includeReviews=true
    }
}
```

### Why API Controllers "Just Work" with JSON

This is why you can send JSON to an API controller action without explicitly adding `[FromBody]`:

```csharp
[ApiController]
[Route("api/orders")]
public class OrdersController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(CreateOrderRequest request)
    {
        // This "just works" with JSON because:
        // 1. CreateOrderRequest is a complex type
        // 2. [ApiController] infers [FromBody] for complex types
        // 3. The default input formatter is System.Text.Json
        return CreatedAtAction(nameof(Get), new { id = request.Id }, request);
    }
    
    [HttpGet("{id}")]
    public IActionResult Get(int id) => Ok();
}
```

### Disabling Inference

If you need to opt out of the automatic inference for a specific parameter, apply an explicit binding source attribute:

```csharp
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    // Override: bind complex type from form instead of body
    [HttpPost]
    public IActionResult Create([FromForm] CreateProductRequest request)
    {
        // Even though [ApiController] would infer [FromBody],
        // the explicit [FromForm] takes precedence
    }
}
```

```ad-attention
When using `[ApiController]`, if you have an action that receives **both** a complex type from the body and a file upload, you need to explicitly mark the complex type as `[FromForm]` because you cannot mix `[FromBody]` and `[FromForm]` -- multipart forms carry both data and files together.
```
