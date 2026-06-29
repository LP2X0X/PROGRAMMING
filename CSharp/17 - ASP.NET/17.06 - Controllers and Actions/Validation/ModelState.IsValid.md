---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


When the model binder finishes binding and validation, it stores all errors in `ModelState` -- a `ModelStateDictionary` on the controller base class.

### Basic Pattern

```csharp
[HttpPost]
public IActionResult CreateProduct(CreateProductRequest request)
{
    if (!ModelState.IsValid)
    {
        // Return the form with validation error messages displayed
        return View(request);
    }

    // Business logic -- only reached if all validation passed
    _productService.Create(request);
    return RedirectToAction(nameof(Index));
}
```

For API controllers returning JSON:

```csharp
[HttpPost]
public IActionResult CreateProduct(CreateProductRequest request)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    var product = _productService.Create(request);
    return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
}
```

### Adding Custom Errors

You can add errors to `ModelState` manually for business-rule violations that go beyond attribute-level validation:

```csharp
[HttpPost]
public IActionResult CreateOrder(CreateOrderRequest request)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    // Business rule: check if the product actually exists
    if (!_productService.Exists(request.ProductId))
    {
        ModelState.AddModelError(nameof(request.ProductId),
            "The specified product does not exist.");
        return BadRequest(ModelState);
    }

    // Model-level error (not tied to a specific property)
    if (_orderService.HasReachedDailyLimit(request.CustomerId))
    {
        ModelState.AddModelError(string.Empty,
            "Daily order limit has been reached for this customer.");
        return BadRequest(ModelState);
    }

    var order = _orderService.Create(request);
    return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
}
```

The first parameter of `AddModelError` is the **key** -- use the property name for property-level errors, or `string.Empty` for model-level errors.

### What the Error Response Looks Like

When you return `BadRequest(ModelState)`, the response body is a JSON dictionary of errors keyed by property name:

```json
{
  "Name": ["The Name field is required."],
  "Price": ["The field Price must be between 0.01 and 99999.99."],
  "ContactEmail": ["The ContactEmail field is not a valid e-mail address."]
}
```

```ad-summary
`ModelState` is a `ModelStateDictionary` collecting all binding and validation errors. Check `ModelState.IsValid` to guard your action logic. Use `AddModelError(key, message)` for custom business-rule errors. Pass `string.Empty` as the key for model-level errors not tied to a specific property.
```
