---
tags:
  - csharp
  - asp-net-core
  - web-api
  - conventions
  - openapi
---


The **`[ProducesResponseType]`** attribute decorates controller actions to declare which HTTP status codes and response body types an action can return. This metadata is consumed by OpenAPI/Swagger tooling to generate accurate API documentation and by analyzers to warn about undocumented responses.

### Basic Syntax

```csharp
[HttpGet("{id}")]
[ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> GetProduct(int id)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null)
        return NotFound();

    return Ok(_mapper.Map<ProductDto>(product));
}
```

### Generic Syntax (.NET 7+)

Starting with .NET 7, a cleaner generic syntax is available:

```csharp
[HttpGet("{id}")]
[ProducesResponseType<ProductDto>(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> GetProduct(int id)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null)
        return NotFound();

    return Ok(_mapper.Map<ProductDto>(product));
}
```

### Common Patterns for CRUD Operations

```csharp
// GET collection
[HttpGet]
[ProducesResponseType<IEnumerable<ProductDto>>(StatusCodes.Status200OK)]
public async Task<IActionResult> GetAll() { /* ... */ }

// GET single
[HttpGet("{id}")]
[ProducesResponseType<ProductDto>(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> GetById(int id) { /* ... */ }

// POST create
[HttpPost]
[ProducesResponseType<ProductDto>(StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public async Task<IActionResult> Create(CreateProductDto dto) { /* ... */ }

// PUT update
[HttpPut("{id}")]
[ProducesResponseType(StatusCodes.Status204NoContent)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> Update(int id, UpdateProductDto dto) { /* ... */ }

// DELETE
[HttpDelete("{id}")]
[ProducesResponseType(StatusCodes.Status204NoContent)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> Delete(int id) { /* ... */ }
```

### Typed Return Values Reduce Boilerplate

When an action returns `ActionResult<T>`, the 200 response type is inferred automatically:

```csharp
[HttpGet("{id}")]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<ActionResult<ProductDto>> GetProduct(int id)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null)
        return NotFound();

    // 200 + ProductDto is inferred from ActionResult<ProductDto>
    return _mapper.Map<ProductDto>(product);
}
```

> [!tip]
> When using `ActionResult<T>`, the `[ProducesResponseType]` for the 200 status code with the `T` type is automatically inferred. You only need to declare non-success response types explicitly.

### Specifying Content Types

You can also declare the content type of the response:

```csharp
[HttpGet("{id}")]
[Produces("application/json")]
[ProducesResponseType<ProductDto>(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> GetProduct(int id) { /* ... */ }
```

The `[Produces]` attribute constrains the response content type and is separate from `[ProducesResponseType]`, which declares the status code and body type.

> [!warning]
> If you declare `[ProducesResponseType]` but your action can actually return status codes you did not declare, Swagger docs will be incomplete and consumers may not handle those cases. The `API1000` analyzer warns about this.

> [!summary] Section Summary
> `[ProducesResponseType]` declares possible HTTP status codes and response body types for each action, feeding Swagger documentation. The generic syntax `[ProducesResponseType<T>(...)]` is preferred in .NET 7+. Using `ActionResult<T>` auto-infers the success response type.
