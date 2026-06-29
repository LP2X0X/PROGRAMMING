---
tags:
  - csharp
  - asp-net-core
  - action-results
  - controllers
---


The ergonomics of `ActionResult<T>` come from two implicit operators defined on the struct:

```csharp
// Simplified -- what the framework defines:
public sealed class ActionResult<TValue>
{
    // Implicit from TValue -- wraps in OkObjectResult
    public static implicit operator ActionResult<TValue>(TValue value);

    // Implicit from ActionResult -- discards T, uses the action result as-is
    public static implicit operator ActionResult<TValue>(ActionResult result);
}
```

This means you can write natural, clean code:

```csharp
[HttpGet("{id}")]
public ActionResult<Product> GetProduct(int id)
{
    var product = _repository.Find(id);

    if (product is null)
        return NotFound();       // ActionResult -> ActionResult<Product>

    return product;              // Product -> ActionResult<Product> (wrapped in Ok)
}
```

### What Happens at Runtime

When you `return product;`:
1. The implicit operator wraps `product` in a new `ActionResult<Product>` that stores the value.
2. The framework detects the value, creates an `OkObjectResult`, and executes it.
3. The response is 200 with the product serialized as JSON.

When you `return NotFound();`:
1. `NotFound()` returns a `NotFoundResult` (which is an `ActionResult`).
2. The implicit operator wraps it in `ActionResult<Product>`, but the `Product` type is irrelevant.
3. The framework detects the `ActionResult`, ignores `T`, and executes the `NotFoundResult`.
4. The response is 404 with no body.

```ad-note
You cannot `return null;` from `ActionResult<T>` when `T` is a reference type. It compiles, but at runtime it produces a 204 No Content response (the framework treats a null value as "nothing to serialize"). If you want a 404, explicitly return `NotFound()`.
```
