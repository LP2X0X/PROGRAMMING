---
tags:
  - csharp
  - asp-net-core
  - web-api
  - controllers
---


**REST (Representational State Transfer)** maps HTTP verbs to CRUD operations on resources. Each resource is identified by a URL, and the verb indicates the operation.

### The Standard Verb-to-CRUD Mapping

| HTTP Verb | CRUD Operation | Route Example | Description |
|---|---|---|---|
| `GET` | Read | `GET /api/products` | List all products |
| `GET` | Read | `GET /api/products/5` | Get product with ID 5 |
| `POST` | Create | `POST /api/products` | Create a new product |
| `PUT` | Update (full) | `PUT /api/products/5` | Replace product 5 entirely |
| `PATCH` | Update (partial) | `PATCH /api/products/5` | Update specific fields of product 5 |
| `DELETE` | Delete | `DELETE /api/products/5` | Delete product 5 |

### HTTP Response Codes by Verb

Each verb has expected response patterns:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // GET /api/products -> 200 OK with list
    [HttpGet]
    public ActionResult<IEnumerable<ProductDto>> GetAll()
    {
        var products = _repository.GetAll();
        return Ok(products);
    }

    // GET /api/products/5 -> 200 OK or 404 Not Found
    [HttpGet("{id}")]
    public ActionResult<ProductDto> GetById(int id)
    {
        var product = _repository.Find(id);
        if (product is null) return NotFound();
        return Ok(product);
    }

    // POST /api/products -> 201 Created with Location header
    [HttpPost]
    public ActionResult<ProductDto> Create(CreateProductDto dto)
    {
        var product = _repository.Add(dto);
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }

    // PUT /api/products/5 -> 204 No Content or 404 Not Found
    [HttpPut("{id}")]
    public IActionResult Update(int id, UpdateProductDto dto)
    {
        var existing = _repository.Find(id);
        if (existing is null) return NotFound();
        
        _repository.Update(id, dto);
        return NoContent();
    }

    // DELETE /api/products/5 -> 204 No Content or 404 Not Found
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        var existing = _repository.Find(id);
        if (existing is null) return NotFound();
        
        _repository.Delete(id);
        return NoContent();
    }
}
```

### PATCH with JSON Patch

**Partial updates** use the `PATCH` verb with a JSON Patch document. Install the `Microsoft.AspNetCore.JsonPatch` and `Microsoft.AspNetCore.Mvc.NewtonsoftJson` NuGet packages:

```bash
dotnet add package Microsoft.AspNetCore.JsonPatch
dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson
```

Configure Newtonsoft.Json (required for JSON Patch support):

```csharp
builder.Services
    .AddControllers()
    .AddNewtonsoftJson();
```

The PATCH action:

```csharp
[HttpPatch("{id}")]
public IActionResult Patch(int id, [FromBody] JsonPatchDocument<UpdateProductDto> patchDoc)
{
    var existing = _repository.Find(id);
    if (existing is null) return NotFound();

    var dto = MapToDto(existing);
    patchDoc.ApplyTo(dto, ModelState);

    if (!TryValidateModel(dto))
        return BadRequest(ModelState);

    _repository.Update(id, dto);
    return NoContent();
}
```

A JSON Patch request body looks like:

```json
[
    { "op": "replace", "path": "/price", "value": 59.99 },
    { "op": "replace", "path": "/name", "value": "Wireless Keyboard" }
]
```

> [!tip]
> The HTTP request for a PATCH operation uses `Content-Type: application/json-patch+json`, not `application/json`.

### Nested Resources

For resources that belong to another resource, use nested routes:

```csharp
[ApiController]
[Route("api/products/{productId}/reviews")]
public class ProductReviewsController : ControllerBase
{
    [HttpGet]                      // GET /api/products/5/reviews
    public IActionResult GetAll(int productId) { ... }

    [HttpGet("{reviewId}")]        // GET /api/products/5/reviews/3
    public IActionResult GetById(int productId, int reviewId) { ... }

    [HttpPost]                     // POST /api/products/5/reviews
    public IActionResult Create(int productId, CreateReviewDto dto) { ... }
}
```

### Idempotency

An important REST principle: ==`GET`, `PUT`, and `DELETE` should be idempotent== (calling them multiple times with the same input produces the same result). `POST` is not idempotent -- each call creates a new resource.

| Verb | Idempotent | Safe (read-only) |
|---|---|---|
| GET | Yes | Yes |
| POST | No | No |
| PUT | Yes | No |
| PATCH | No* | No |
| DELETE | Yes | No |

*PATCH can be idempotent depending on the operations in the patch document.

> [!summary] Section Summary
> RESTful APIs map HTTP verbs to CRUD operations: GET for reading, POST for creating, PUT for full updates, PATCH for partial updates, and DELETE for removal. Each verb has standard response codes. GET, PUT, and DELETE are idempotent. Nested routes handle sub-resources.
