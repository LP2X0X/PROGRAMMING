---
tags:
  - csharp
  - asp-net-core
  - controllers
  - mvc
---


The `[ApiController]` attribute activates a set of opinionated conventions that make API development cleaner and more consistent. Apply it at the controller class level.

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // ...
}
```

### What It Enables

#### 1. Automatic Model Validation

Without `[ApiController]`, you must manually check `ModelState`:

```csharp
// WITHOUT [ApiController] -- manual validation
[HttpPost]
public IActionResult Create(ProductCreateDto dto)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    var product = _productService.Create(dto);
    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
}
```

With `[ApiController]`, the framework does this automatically. If `ModelState` is invalid, it short-circuits and returns a 400 with `ProblemDetails` before your action code ever runs:

```csharp
// WITH [ApiController] -- no manual check needed
[HttpPost]
public IActionResult Create(ProductCreateDto dto)
{
    // If we reach here, ModelState is guaranteed valid
    var product = _productService.Create(dto);
    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
}
```

See [[Validation]] for details on data annotations and custom validators.

#### 2. Binding Source Inference

The framework infers where to bind action parameters from:

| Parameter Type | Inferred Source |
|---|---|
| Complex types (classes) | `[FromBody]` |
| `IFormFile` / `IFormFileCollection` | `[FromForm]` |
| Route parameter match | `[FromRoute]` |
| Everything else | `[FromQuery]` |

Without `[ApiController]`, you would need to explicitly decorate every parameter. See [[Model Binding]] for full details.

#### 3. ProblemDetails Error Responses

Error responses automatically follow the **RFC 7807** `ProblemDetails` format:

```json
{
    "type": "https://tools.ietf.org/html/rfc7807",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "traceId": "00-abc123...",
    "errors": {
        "Name": ["The Name field is required."],
        "Price": ["The field Price must be between 0.01 and 10000."]
    }
}
```

#### 4. Attribute Routing Required

When `[ApiController]` is applied, **convention-based routing does not work**. You must use attribute routing (`[Route]`, `[HttpGet]`, etc.). This is intentional -- API endpoints should have explicit, predictable URLs. See [[17.05 - Routing]].

```ad-warning
If you forget to add `[Route]` to an `[ApiController]` class, the app will throw an exception at startup telling you that attribute routing is required.
```
