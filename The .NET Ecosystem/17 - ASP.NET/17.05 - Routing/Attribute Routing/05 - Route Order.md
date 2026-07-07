---
tags:
  - csharp
  - asp-net-core
  - routing
  - attribute-routing
---


The `Order` property on route attributes controls the evaluation priority when multiple routes could match.

### How Order Works

```csharp
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet("special", Order = 0)]   // Evaluated first
    public IActionResult Special() => Ok();

    [HttpGet("{name}", Order = 1)]    // Evaluated second
    public IActionResult GetByName(string name) => Ok();
}
```

- **Lower `Order` values are evaluated first** (higher priority)
- The **default `Order` is `0`**
- Routes with the same `Order` are ranked by specificity (literals beat parameters)

### When Order Matters

Order is rarely needed because the default specificity rules handle most cases. The literal route `"special"` naturally wins over the parameter route `"{name}"` without any `Order` setting.

Order becomes useful when:
- Two routes have ambiguous specificity
- You want to explicitly control which route "wins" for overlapping patterns
- You are combining routes from multiple controllers that might conflict

> [!tip] Practical Tip
> If you find yourself setting `Order` frequently, step back and reconsider your URL design. Well-designed routes rarely need explicit ordering. The specificity algorithm handles the vast majority of cases correctly.

> [!summary] Section Summary
> - `Order` controls route evaluation priority -- lower values are checked first.
> - Default `Order` is `0`; routes with the same order are ranked by specificity.
> - Explicit ordering is rarely needed; prefer designing routes with clear specificity differences.
