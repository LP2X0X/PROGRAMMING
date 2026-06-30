---
tags:
  - csharp
  - asp-net-core
  - routing
  - attribute-routing
---


**Attribute routing** is a routing mechanism where URL patterns are declared directly on controller classes and action methods using C# attributes. Instead of defining routes in a centralized route table, each controller and action carries its own route definition.

```csharp
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [HttpGet]           // GET api/products
    public IActionResult GetAll() => Ok();

    [HttpGet("{id}")]   // GET api/products/5
    public IActionResult GetById(int id) => Ok();
}
```

### Key Advantages

- **Self-documenting**: Looking at a controller immediately tells you its URLs
- **Explicit**: No convention magic -- every route is declared
- **Flexible**: Full control over URL structure without fighting conventions
- **Required for `[ApiController]`**: The `[ApiController]` attribute mandates attribute routing

> [!ad-note] Key Insight
> When you apply `[ApiController]` to a controller, ASP.NET Core **requires** that all actions use attribute routing. Conventional routing will not work. This is by design -- APIs need explicit, predictable URL structures.

> [!summary] Section Summary
> - Attribute routing declares URL patterns on controllers and actions via C# attributes.
> - It is self-documenting, explicit, and required for `[ApiController]`-decorated controllers.
> - Each action explicitly declares which URLs and HTTP methods it handles.
