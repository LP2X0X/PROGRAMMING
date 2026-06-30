---
tags:
  - csharp
  - asp-net-core
  - web-api
  - conventions
  - openapi
---


Manually annotating every action with `[ProducesResponseType]` is repetitive. **`[ApiConventionMethod]`** and **`[ApiConventionType]`** let you apply response type conventions in bulk by matching action signatures to convention methods.

### ApiConventionType -- Apply to a Controller

```csharp
[ApiController]
[Route("api/[controller]")]
[ApiConventionType(typeof(DefaultApiConventions))]
public class ProductsController : ControllerBase
{
    // All actions in this controller inherit response type
    // metadata from DefaultApiConventions based on method
    // name and parameter matching.

    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> Get(int id) { /* ... */ }

    [HttpPost]
    public async Task<ActionResult<ProductDto>> Post(CreateProductDto dto) { /* ... */ }

    [HttpPut("{id}")]
    public async Task<IActionResult> Put(int id, UpdateProductDto dto) { /* ... */ }

    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id) { /* ... */ }
}
```

### ApiConventionMethod -- Apply to a Single Action

When you only want to apply conventions to a specific action, or when the default name matching does not apply:

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpPut("{id}/cancel")]
    [ApiConventionMethod(typeof(DefaultApiConventions), nameof(DefaultApiConventions.Put))]
    public async Task<IActionResult> CancelOrder(int id)
    {
        // This action doesn't follow the "Put" naming convention,
        // but we tell ASP.NET to treat it like a Put for conventions.
        var order = await _repository.GetByIdAsync(id);
        if (order is null) return NotFound();

        order.Status = OrderStatus.Cancelled;
        await _repository.UpdateAsync(order);
        return NoContent();
    }
}
```

### Assembly-Level Convention

You can apply conventions to every controller in the assembly:

```csharp
// In Program.cs or a dedicated AssemblyInfo.cs
[assembly: ApiConventionType(typeof(DefaultApiConventions))]
```

> [!ad-note]
> The convention matching is based on method names and parameter names. A method named `Get` with an `id` parameter matches `DefaultApiConventions.Get(int id)`. If your naming deviates, use `[ApiConventionMethod]` explicitly.

### Creating Custom Conventions

You can define your own convention type with custom response type metadata:

```csharp
public static class CustomApiConventions
{
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesDefaultResponseType]
    public static void Get(
        [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Suffix)]
        int id)
    { }

    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesDefaultResponseType]
    public static void Create(
        [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Any)]
        object model)
    { }

    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesDefaultResponseType]
    public static void Update(
        [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Suffix)]
        int id,
        [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Any)]
        object model)
    { }

    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesDefaultResponseType]
    public static void Delete(
        [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Suffix)]
        int id)
    { }
}
```

The `ApiConventionNameMatchBehavior` enum controls how parameter names are matched:

| Behavior | Description |
|---|---|
| `Any` | Matches any parameter name |
| `Exact` | Must match exactly |
| `Prefix` | Must start with the convention parameter name |
| `Suffix` | Must end with the convention parameter name |

Apply your custom conventions:

```csharp
[ApiConventionType(typeof(CustomApiConventions))]
public class ProductsController : ControllerBase { /* ... */ }
```

> [!summary] Section Summary
> `[ApiConventionType]` applies response type conventions to an entire controller or assembly. `[ApiConventionMethod]` targets individual actions. Custom conventions let you define project-specific response patterns with flexible name matching.
