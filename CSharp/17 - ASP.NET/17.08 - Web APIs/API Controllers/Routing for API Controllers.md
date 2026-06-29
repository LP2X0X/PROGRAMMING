---
tags:
  - csharp
  - asp-net-core
  - web-api
  - controllers
---


**Attribute routing** is the standard approach for API controllers. Convention-based routing (used in MVC) is not recommended for APIs.

### The Standard Route Template

```csharp
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // [controller] is replaced with "products" (class name minus "Controller" suffix)
}
```

The **`[controller]`** token is a route token replacement. For `ProductsController`, it becomes `products`. For `OrderItemsController`, it becomes `orderitems`.

### Action-Level Route Templates

Each action method specifies its HTTP verb and optional route extension:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]                  // GET api/products
    public IActionResult GetAll() { ... }

    [HttpGet("{id}")]          // GET api/products/5
    public IActionResult GetById(int id) { ... }

    [HttpGet("{id}/reviews")]  // GET api/products/5/reviews
    public IActionResult GetReviews(int id) { ... }

    [HttpPost]                 // POST api/products
    public IActionResult Create(CreateProductDto dto) { ... }

    [HttpPut("{id}")]          // PUT api/products/5
    public IActionResult Update(int id, UpdateProductDto dto) { ... }

    [HttpDelete("{id}")]       // DELETE api/products/5
    public IActionResult Delete(int id) { ... }
}
```

### Route Constraints

Route constraints restrict which values match a route parameter:

```csharp
[HttpGet("{id:int}")]                    // Only matches integer values
[HttpGet("{id:int:min(1)}")]             // Integer >= 1
[HttpGet("{slug:alpha}")]                // Only alphabetic characters
[HttpGet("{date:datetime}")]             // Valid DateTime values
[HttpGet("{id:guid}")]                   // Valid GUID values
[HttpGet("{name:minlength(3)}")]         // At least 3 characters
[HttpGet("{name:maxlength(50)}")]        // At most 50 characters
[HttpGet("{name:regex(^[a-z]+$)}")]      // Matches regex pattern
```

Multiple constraints can be combined:

```csharp
[HttpGet("{id:int:min(1):max(999)}")]    // Integer between 1 and 999
```

### Naming Routes for Link Generation

Named routes allow you to generate URLs to specific actions:

```csharp
[HttpGet("{id}", Name = "GetProductById")]
public IActionResult GetById(int id)
{
    var product = _repository.Find(id);
    if (product is null) return NotFound();
    return Ok(product);
}

[HttpPost]
public IActionResult Create(CreateProductDto dto)
{
    var product = _repository.Add(dto);
    
    // Generates a URL like "/api/products/42"
    return CreatedAtRoute("GetProductById", new { id = product.Id }, product);
}
```

> [!ad-note]
> You can also use `CreatedAtAction` which references the action method name instead of a route name:
> ```csharp
> return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
> ```

> [!summary] Section Summary
> API controllers use attribute routing with `[Route("api/[controller]")]` at the class level and HTTP verb attributes (`[HttpGet]`, `[HttpPost]`, etc.) at the action level. Route constraints validate parameters, and named routes enable URL generation for `CreatedAtRoute` responses.
