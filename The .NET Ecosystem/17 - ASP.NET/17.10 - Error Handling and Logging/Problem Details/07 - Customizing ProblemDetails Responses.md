---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---


Beyond global configuration, you can customize ProblemDetails at the point of creation for specific error scenarios.

## In Controllers

```csharp
[HttpGet("{id}")]
public IActionResult GetProduct(int id)
{
    var product = _repository.GetById(id);

    if (product is null)
    {
        return Problem(
            type: "https://example.com/errors/product-not-found",
            title: "Product Not Found",
            statusCode: StatusCodes.Status404NotFound,
            detail: $"No product exists with ID {id}.",
            instance: HttpContext.Request.Path
        );
    }

    return Ok(product);
}
```

## Creating ProblemDetails Objects Directly

```csharp
[HttpDelete("{id}")]
public IActionResult DeleteProduct(int id)
{
    var product = _repository.GetById(id);

    if (product is null)
        return NotFound();

    if (product.HasActiveOrders)
    {
        var problemDetails = new ProblemDetails
        {
            Type = "https://example.com/errors/product-has-orders",
            Title = "Cannot Delete Product",
            Status = StatusCodes.Status409Conflict,
            Detail = $"Product '{product.Name}' has {product.ActiveOrderCount} " +
                     "active orders and cannot be deleted.",
            Instance = HttpContext.Request.Path
        };

        problemDetails.Extensions["productId"] = id;
        problemDetails.Extensions["activeOrderCount"] = product.ActiveOrderCount;

        return new ObjectResult(problemDetails)
        {
            StatusCode = StatusCodes.Status409Conflict,
            ContentTypes = { "application/problem+json" }
        };
    }

    _repository.Delete(id);
    return NoContent();
}
```

> [!summary] Section Summary
> - Use the `Problem()` helper method on `ControllerBase` for quick ProblemDetails responses
> - Create `ProblemDetails` objects directly when you need to add extension members
> - Set `ContentTypes` to `"application/problem+json"` on the `ObjectResult` for correct media type
> - Include relevant context in extension members (IDs, counts, timestamps) that help clients handle the error
