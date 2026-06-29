---
tags:
  - csharp
  - asp-net-core
  - controllers
  - mvc
---


ASP.NET Core is built on an asynchronous pipeline. Actions that perform I/O (database calls, HTTP calls, file reads) should **always** be async.

### Sync vs Async

```csharp
// BAD -- blocks a thread pool thread while waiting for the database
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var product = _productRepo.GetById(id);  // synchronous -- thread is blocked
    if (product is null)
        return NotFound();

    return Ok(product);
}

// GOOD -- frees the thread while waiting for the database
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id)
{
    var product = await _productRepo.GetByIdAsync(id);  // thread returns to pool
    if (product is null)
        return NotFound();

    return Ok(product);
}
```

```ad-danger
**Never use `Task.Run()` in ASP.NET Core actions.** ASP.NET Core shares its thread pool with the request pipeline. Wrapping synchronous code in `Task.Run()` does not help -- it just offloads work to another thread pool thread while the original thread waits. You end up using **two** threads instead of one, reducing overall throughput.
```

### Cancellation Tokens

ASP.NET Core automatically binds a `CancellationToken` parameter to the request's abort token. If the client disconnects, the token is cancelled, and your async operation can stop early:

```csharp
[HttpGet]
public async Task<IActionResult> GetAll(CancellationToken cancellationToken)
{
    // If the client disconnects, this query is cancelled
    var products = await _dbContext.Products
        .AsNoTracking()
        .ToListAsync(cancellationToken);

    return Ok(products);
}
```

```ad-tip
Always pass the `CancellationToken` down to every async call in the chain (EF Core queries, `HttpClient` calls, etc.). This ensures resources are freed promptly when clients abandon requests.
```

### Strongly-Typed Return Values

Instead of `Task<IActionResult>`, you can use `Task<ActionResult<T>>` for better Swagger/OpenAPI documentation:

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<ProductDto>> GetById(int id)
{
    var product = await _productRepo.GetByIdAsync(id);
    if (product is null)
        return NotFound();  // implicitly converts to ActionResult<ProductDto>

    return product;  // implicitly wraps in Ok(product)
}
```
