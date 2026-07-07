---
tags:
  - csharp
  - asp-net-core
  - action-results
  - controllers
---


There are three ways to declare an action method's return type. Each has its place:

### Return T Directly

Use when the action always succeeds with 200. Rare in practice because most endpoints need error handling.

```csharp
[HttpGet]
public IEnumerable<Product> GetAllProducts()
{
    return _repository.GetAll();
}
```

If the method throws, the exception middleware handles it. There is no way to return a 404 or 400 from this signature without throwing.

### Return IActionResult

Use when you need multiple status codes and don't care about strong typing on the success path.

```csharp
[HttpGet("{id}")]
public IActionResult GetProduct(int id)
{
    var product = _repository.Find(id);

    if (product is null)
        return NotFound();

    return Ok(product);
}
```

Works, but Swagger cannot infer the success response type without `[ProducesResponseType]` attributes.

### Return ActionResult\<T\> (Recommended for APIs)

Use when you want both multiple status codes and strong typing.

```csharp
[HttpGet("{id}")]
public ActionResult<Product> GetProduct(int id)
{
    var product = _repository.Find(id);

    if (product is null)
        return NotFound();

    return product;    // implicit conversion, auto-wrapped in Ok
}
```

```ad-tip
**Rule of thumb for APIs:**
- Default to `ActionResult<T>` for most endpoints.
- Use `IActionResult` when the success type genuinely varies (e.g., file downloads that might also return JSON errors).
- Use `T` directly only for trivial endpoints that never fail.
```

### Async Actions

All three approaches work with `Task<>`:

```csharp
// Concrete type
public async Task<Product> GetProduct(int id) { ... }

// IActionResult
public async Task<IActionResult> GetProduct(int id) { ... }

// ActionResult<T> (recommended)
public async Task<ActionResult<Product>> GetProduct(int id) { ... }
```
