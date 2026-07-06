---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---


A common question is whether you should use `try-catch` blocks inside controller actions. The short answer: **rarely**.

## When NOT to Use try-catch in Controllers

For most exceptions, let them propagate to the global exception handling middleware. Catching exceptions in every controller leads to duplicated error handling code and inconsistent responses.

```csharp
// BAD: Duplicated error handling in every action
[HttpGet("{id}")]
public async Task<IActionResult> GetProduct(int id)
{
    try
    {
        var product = await _productService.GetByIdAsync(id);
        return Ok(product);
    }
    catch (NotFoundException ex)
    {
        return NotFound(new { error = ex.Message });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error getting product {Id}", id);
        return StatusCode(500, new { error = "Internal error" });
    }
}

// GOOD: Let the global middleware handle it
[HttpGet("{id}")]
public async Task<IActionResult> GetProduct(int id)
{
    var product = await _productService.GetByIdAsync(id);
    return Ok(product);
    // If GetByIdAsync throws NotFoundException, the middleware maps it to 404
    // If it throws anything else, the middleware logs it and returns 500
}
```

## When try-catch IS Appropriate

Use `try-catch` in controllers when you need to **recover** from the error and continue with an alternative path, not just translate it into a different error format:

```csharp
[HttpPost("import")]
public async Task<IActionResult> ImportProducts(IFormFile file)
{
    var results = new List<ImportResult>();

    foreach (var row in ParseCsv(file))
    {
        try
        {
            // Try to import each row individually
            await _productService.CreateAsync(row);
            results.Add(new ImportResult(row.Name, Success: true));
        }
        catch (ValidationException ex)
        {
            // One bad row should not abort the entire import
            results.Add(new ImportResult(row.Name, Success: false,
                Errors: ex.Errors));
        }
        // Let other exceptions (DB down, etc.) propagate to middleware
    }

    return Ok(new { imported = results.Count(r => r.Success), results });
}
```

```csharp
// Another valid case: calling an external service with a fallback
[HttpGet("{id}")]
public async Task<IActionResult> GetProductWithReviews(int id)
{
    var product = await _productService.GetByIdAsync(id);

    try
    {
        // External review service might be down
        product.Reviews = await _reviewService.GetForProductAsync(id);
    }
    catch (HttpRequestException)
    {
        // Degrade gracefully -- return the product without reviews
        product.Reviews = Array.Empty<Review>();
        _logger.LogWarning("Review service unavailable for product {Id}", id);
    }

    return Ok(product);
}
```

> [!tip]
> **Rule of thumb:** If the catch block translates the exception into an error response, that logic belongs in middleware or an `IExceptionHandler`. If the catch block provides a **fallback behavior** and the request still succeeds, it belongs in the controller.

> [!summary] Section Summary
> - Avoid try-catch in controllers when the goal is just to format error responses -- that is the middleware's job
> - Use try-catch when you need to recover and continue (partial imports, graceful degradation with fallbacks)
> - Letting exceptions propagate to global handlers keeps controller code clean and error handling consistent
> - The exception type should carry enough information for the middleware to generate the right response
