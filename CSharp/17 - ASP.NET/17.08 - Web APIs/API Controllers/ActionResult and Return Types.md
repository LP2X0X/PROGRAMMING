---
tags:
  - csharp
  - asp-net-core
  - web-api
  - controllers
---


Choosing the right return type for your action methods affects both runtime behavior and API documentation (Swagger/OpenAPI).

### The Three Return Type Options

#### 1. Specific Type (Direct Return)

```csharp
[HttpGet]
public IEnumerable<ProductDto> GetAll()
{
    return _repository.GetAll();
}
```

- Always returns 200 OK
- Cannot return different status codes (404, 400, etc.)
- Limited Swagger documentation

#### 2. `IActionResult` (Maximum Flexibility)

```csharp
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var product = _repository.Find(id);
    if (product is null) return NotFound();
    return Ok(product);
}
```

- Can return any status code
- Swagger cannot infer the response type without explicit `[ProducesResponseType]`

#### 3. `ActionResult<T>` (Best of Both Worlds)

```csharp
[HttpGet("{id}")]
public ActionResult<ProductDto> GetById(int id)
{
    var product = _repository.Find(id);
    if (product is null) return NotFound();
    return product;  // Implicit conversion to ActionResult<ProductDto>
}
```

- ==This is the recommended return type for API actions==
- Swagger automatically infers the `200 OK` response type as `ProductDto`
- Can still return different status codes
- Supports implicit conversion from `T` (no need for `Ok(product)`)

### Documenting Response Types with `[ProducesResponseType]`

For complete OpenAPI documentation, annotate actions with the status codes they can return:

```csharp
[HttpGet("{id}")]
[ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public ActionResult<ProductDto> GetById(int id)
{
    var product = _repository.Find(id);
    if (product is null) return NotFound();
    return product;
}

[HttpPost]
[ProducesResponseType(typeof(ProductDto), StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public ActionResult<ProductDto> Create(CreateProductDto dto)
{
    var product = _repository.Add(dto);
    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
}
```

> [!tip]
> With `ActionResult<T>`, you can use the shorter form for the success response:
> ```csharp
> [ProducesResponseType(StatusCodes.Status200OK)]      // Type inferred from ActionResult<T>
> [ProducesResponseType(StatusCodes.Status404NotFound)]
> public ActionResult<ProductDto> GetById(int id) { ... }
> ```

### Controller-Level Response Type Annotations

Apply shared response types at the controller level with `[Produces]`:

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]                          // All actions produce JSON
[ProducesResponseType(StatusCodes.Status500InternalServerError)]  // All actions may 500
public class ProductsController : ControllerBase
{
    // Individual actions only need their specific annotations
}
```

See [[API Conventions]] for applying conventions across controllers.

> [!summary] Section Summary
> Use `ActionResult<T>` as the return type for API actions -- it combines type safety, flexible status code returns, and automatic Swagger/OpenAPI documentation. Annotate actions with `[ProducesResponseType]` to document all possible response codes.
