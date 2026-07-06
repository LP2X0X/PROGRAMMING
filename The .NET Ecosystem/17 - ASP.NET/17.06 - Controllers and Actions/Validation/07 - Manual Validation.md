---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


Sometimes you need to validate a model that was not bound from an HTTP request -- for example, a model constructed in code or loaded from a database.

### TryValidateModel (Inside Controllers)

```csharp
[HttpPost]
public IActionResult ImportProducts(IFormFile file)
{
    var products = _csvParser.Parse<CreateProductRequest>(file);

    foreach (var product in products)
    {
        // Clear previous errors and validate this model
        ModelState.Clear();

        if (!TryValidateModel(product))
        {
            return BadRequest(new
            {
                Error = $"Validation failed for product: {product.Name}",
                Details = ModelState
            });
        }
    }

    _productService.BulkCreate(products);
    return Ok();
}
```

### Validator.TryValidateObject (Outside Controllers)

For validation outside the controller pipeline (services, background jobs, console apps):

```csharp
using System.ComponentModel.DataAnnotations;

public static class ValidationHelper
{
    public static (bool IsValid, List<ValidationResult> Errors) Validate<T>(T model)
        where T : notnull
    {
        var context = new ValidationContext(model);
        var results = new List<ValidationResult>();

        // The 'true' parameter validates ALL properties, not just [Required]
        bool isValid = Validator.TryValidateObject(
            model, context, results, validateAllProperties: true);

        return (isValid, results);
    }
}

// Usage in a background service
public class OrderProcessingService
{
    public void ProcessOrder(CreateOrderRequest request)
    {
        var (isValid, errors) = ValidationHelper.Validate(request);

        if (!isValid)
        {
            throw new ValidationException(
                string.Join("; ", errors.Select(e => e.ErrorMessage)));
        }

        // Process the valid order...
    }
}
```

```ad-attention
The `validateAllProperties` parameter (the last `bool` argument) defaults to `false` if omitted. When `false`, only `[Required]` attributes are checked. **Always pass `true`** to validate all Data Annotation attributes on the object.
```

```ad-summary
Use `TryValidateModel` inside controllers when you construct models in code. Use `Validator.TryValidateObject` outside the controller pipeline (services, background jobs). Always pass `validateAllProperties: true` to check all attributes, not just `[Required]`.
```
