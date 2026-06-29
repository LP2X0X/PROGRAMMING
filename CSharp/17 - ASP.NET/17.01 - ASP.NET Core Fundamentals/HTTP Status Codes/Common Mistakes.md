---
tags:
  - csharp
  - asp-net-core
  - http
  - status-codes
  - web-api
---

## Common Mistakes

### Mistake 1: Confusing 401 and 403

As covered above, 401 means "not authenticated" and 403 means "not authorized." Because 401 is named "Unauthorized," developers frequently mix them up.

```csharp
// WRONG -- returning 403 when the user is not logged in
if (User.Identity?.IsAuthenticated != true)
    return Forbid(); // 403 -- but this should be 401!

// CORRECT
if (User.Identity?.IsAuthenticated != true)
    return Unauthorized(); // 401 -- not authenticated

if (!User.IsInRole("Admin"))
    return Forbid(); // 403 -- authenticated but not authorized
```

### Mistake 2: Returning 200 with an Error Message

This is common among developers new to APIs. They return 200 OK for every response and put the error in the body.

```csharp
// WRONG -- 200 with error in body
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var product = _repo.GetById(id);
    if (product is null)
        return Ok(new { success = false, error = "Product not found" }); // 200!

    return Ok(new { success = true, data = product });
}

// CORRECT -- use proper status codes
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var product = _repo.GetById(id);
    if (product is null)
        return NotFound(); // 404 -- the right status code

    return Ok(product);    // 200
}
```

> [!ad-warning] Why This Is Harmful
> Returning 200 for errors breaks HTTP semantics. Tools like Swagger, `HttpClient`, browser DevTools, and monitoring dashboards all rely on status codes to categorize responses. If everything is 200, you cannot distinguish success from failure without parsing the body -- and monitoring tools will report 100% success even when half your requests are failing.

### Mistake 3: Returning 500 for Validation Errors

If the client sends bad data, that is the client's fault -- a 4xx code. A 500 means the server has a bug.

```csharp
// WRONG -- validation error → 500
[HttpPost]
public IActionResult Create(CreateProductDto dto)
{
    if (dto.Price < 0)
        throw new ArgumentException("Price must be positive");
        // Unhandled → exception middleware → 500 Internal Server Error

    // ...
}

// CORRECT -- validation error → 400 or 422
[HttpPost]
public IActionResult Create(CreateProductDto dto)
{
    if (dto.Price < 0)
        return BadRequest(new { error = "Price must be positive" }); // 400

    // ...
}
```

### Mistake 4: Not Returning a Location Header with 201

When you return `201 Created`, the HTTP spec says you should include a `Location` header pointing to the newly created resource. Using `CreatedAtAction()` or `CreatedAtRoute()` does this automatically.

```csharp
// WRONG -- 201 without Location header
return StatusCode(201, product);

// CORRECT -- 201 with Location header
return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
// Response: 201 Created
// Location: /api/products/42
```

### Mistake 5: Using 404 When the Collection Is Empty

An empty collection is not "not found." The resource (the collection endpoint) exists; it just has no items.

```csharp
// WRONG -- empty list → 404
[HttpGet]
public IActionResult GetAll()
{
    var products = _repo.GetAll();
    if (!products.Any())
        return NotFound(); // 404 -- but the endpoint exists!

    return Ok(products);
}

// CORRECT -- empty list → 200 with empty array
[HttpGet]
public IActionResult GetAll()
{
    var products = _repo.GetAll();
    return Ok(products); // 200 + [] -- the resource exists, it's just empty
}
```

> [!summary] Section Summary
> - 401 = not authenticated, 403 = not authorized -- the naming is misleading
> - Never return 200 with an error message in the body -- use proper status codes
> - Never return 500 for validation errors -- that is a 400 or 422
> - Use `CreatedAtAction()` with 201 to include the `Location` header automatically
> - An empty collection is 200 with an empty array, not 404
