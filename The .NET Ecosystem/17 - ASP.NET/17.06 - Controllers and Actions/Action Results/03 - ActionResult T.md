---
tags:
  - csharp
  - asp-net-core
  - action-results
  - controllers
---


`ActionResult<T>` combines the flexibility of `IActionResult` (returning different status codes) with strong typing on the success path.

### Why It Exists

With plain `IActionResult`, the framework has no idea what type the success response contains. This matters for:

- **Swagger/OpenAPI generation** -- tools like Swashbuckle need to know the response schema
- **Compile-time safety** -- you get no type checking on what you pass to `Ok()`

`ActionResult<T>` solves both problems.

### How It Works

`ActionResult<T>` supports implicit conversion from two sources:

1. **From `T` directly** -- auto-wrapped in an `OkObjectResult` (200)
2. **From any `ActionResult`** -- used as-is (e.g., `NotFound()`, `BadRequest()`)

```csharp
public class ProductsController : ControllerBase
{
    private readonly IProductRepository _repository;

    public ProductsController(IProductRepository repository)
    {
        _repository = repository;
    }

    [HttpGet("{id}")]
    public ActionResult<Product> GetProduct(int id)
    {
        var product = _repository.Find(id);

        if (product is null)
            return NotFound();   // implicit conversion from ActionResult

        return product;          // implicit conversion from T (auto-wrapped in Ok)
    }
}
```

```ad-tip
`ActionResult<T>` is the recommended return type for API actions. It gives you the best of both worlds: multiple status codes and strong typing for the success path.
```

### OpenAPI Benefits

When you use `ActionResult<Product>`, the framework automatically produces `[ProducesResponseType(typeof(Product), 200)]` metadata. Swagger picks this up without you needing to annotate every action:

```csharp
// With IActionResult, Swagger doesn't know the success type:
[HttpGet("{id}")]
[ProducesResponseType(typeof(Product), 200)]   // you must add this manually
[ProducesResponseType(404)]
public IActionResult GetProduct(int id) { ... }

// With ActionResult<T>, Swagger infers the 200 response type:
[HttpGet("{id}")]
[ProducesResponseType(404)]                     // only need to declare non-success codes
public ActionResult<Product> GetProduct(int id) { ... }
```
