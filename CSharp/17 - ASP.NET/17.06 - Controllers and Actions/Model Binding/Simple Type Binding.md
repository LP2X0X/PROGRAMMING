---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


Simple types are scalar values: `int`, `string`, `bool`, `DateTime`, `Guid`, `decimal`, `double`, `enum`, and others that have a `TypeConverter` capable of converting from a string.

### How Conversion Works

All HTTP request data arrives as strings. The model binder uses `TypeConverter` to convert the string representation to the target CLR type:

```csharp
[HttpGet("products")]
public IActionResult Search(
    string name,           // No conversion needed -- already a string
    int page,              // "2" -> 2
    decimal minPrice,      // "19.99" -> 19.99m
    bool inStock,          // "true" -> true (also accepts "True", "TRUE")
    DateTime since,        // "2024-01-15" -> DateTime(2024, 1, 15)
    Guid correlationId,    // "a1b2c3d4-..." -> Guid
    SortOrder sort)        // "Descending" -> SortOrder.Descending (enum)
{
    // All parameters automatically converted from query string values
}

public enum SortOrder
{
    Ascending,
    Descending
}
```

### Special Types

Some types receive special treatment from the binder:

```csharp
[HttpGet("products")]
public async Task<IActionResult> Search(
    string term,
    CancellationToken cancellationToken)  // Automatically bound to HttpContext.RequestAborted
{
    var products = await _productService.SearchAsync(term, cancellationToken);
    return Ok(products);
}
```

`CancellationToken` parameters are automatically bound to `HttpContext.RequestAborted` -- they do not come from request data.

### Conversion Failures

When a string value cannot be converted to the target type, the binder records the error:

```csharp
[HttpGet("products/{id}")]
public IActionResult GetProduct(int id)
{
    // GET /products/abc
    // Conversion from "abc" to int fails:
    // - id = 0 (default)
    // - ModelState["id"].Errors contains:
    //   "The value 'abc' is not valid for id."
}
```

### Nullable Types

Nullable types allow binding to succeed even when the value is absent:

```csharp
[HttpGet("products")]
public IActionResult Search(
    string? term,          // null if not provided
    int? page,             // null if not provided (vs. 0 for non-nullable)
    decimal? maxPrice)     // null if not provided
{
    int actualPage = page ?? 1;        // Default to 1 if not provided
    // term can be checked for null to determine if filtering is requested
}
```
