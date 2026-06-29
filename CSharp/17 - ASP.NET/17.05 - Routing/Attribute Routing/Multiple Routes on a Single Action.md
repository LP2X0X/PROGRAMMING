---
tags:
  - csharp
  - asp-net-core
  - routing
  - attribute-routing
---


A single action can respond to **multiple different URLs** by stacking route attributes:

```csharp
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet("")]
    [HttpGet("all")]
    [HttpGet("list")]
    public IActionResult GetAll() => Ok();
    // Matches: GET api/Products, GET api/Products/all, GET api/Products/list
}
```

You can also stack `[Route]` attributes on the controller:

```csharp
[Route("api/products")]
[Route("api/items")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetById(int id) => Ok();
    // Matches: GET api/products/5 AND GET api/items/5
}
```

### Route Combinatorics

When both controller and action have multiple routes, the result is the **Cartesian product**:

```csharp
[Route("api/products")]
[Route("api/items")]
public class ProductsController : ControllerBase
{
    [HttpGet("")]
    [HttpGet("all")]
    public IActionResult GetAll() => Ok();
    // Produces 4 routes:
    //   GET api/products
    //   GET api/products/all
    //   GET api/items
    //   GET api/items/all
}
```

> [!warning] Common Misconception
> Multiple routes on the same action produce **independent** route entries. Each one is a separate match candidate. If one matches a request, the others are irrelevant for that request. This also means each route can have its own name (for URL generation).

> [!summary] Section Summary
> - Stack multiple route attributes to make one action respond to multiple URLs.
> - Controller and action routes combine as a Cartesian product.
> - Each resulting route is independent and can have its own name.
> - Use this for backward compatibility (old URL + new URL both work) or API aliases.
