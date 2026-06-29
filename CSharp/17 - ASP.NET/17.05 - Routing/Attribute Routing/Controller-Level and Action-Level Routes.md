---
tags:
  - csharp
  - asp-net-core
  - routing
  - attribute-routing
---


Attribute routing uses a **composition model** where controller-level and action-level routes combine.

### Controller-Level Prefix

The `[Route]` attribute on a controller sets a **prefix** for all actions inside it:

```csharp
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [HttpGet]               // GET api/products
    public IActionResult GetAll() => Ok();

    [HttpGet("{id}")]       // GET api/products/{id}
    public IActionResult GetById(int id) => Ok();

    [HttpGet("featured")]   // GET api/products/featured
    public IActionResult Featured() => Ok();
}
```

The controller template `"api/products"` is prepended to each action template.

### Overriding the Controller Prefix

If an action template starts with `/` or `~/`, it **overrides** the controller-level prefix entirely:

```csharp
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [HttpGet]                   // GET api/products
    public IActionResult GetAll() => Ok();

    [HttpGet("/api/featured")]  // GET api/featured  (prefix ignored!)
    public IActionResult Featured() => Ok();
}
```

> [!warning] Common Misconception
> The `/` override is easy to trigger accidentally. If you add a leading slash to an action template thinking it is just "being explicit," you will bypass the controller prefix entirely. Only use a leading `/` when you intentionally want a different base path.

### No Controller-Level Route

If a controller has no `[Route]` attribute, each action must specify its complete path:

```csharp
public class ProductsController : ControllerBase
{
    [HttpGet("api/products")]         // Full path required
    public IActionResult GetAll() => Ok();

    [HttpGet("api/products/{id}")]    // Full path required
    public IActionResult GetById(int id) => Ok();
}
```

This is verbose and error-prone. Always use a controller-level `[Route]`.

> [!summary] Section Summary
> - Controller `[Route]` sets a prefix; action templates are appended to it.
> - Action templates starting with `/` or `~/` override the controller prefix entirely.
> - Omitting the controller `[Route]` forces every action to specify its full path -- avoid this.
> - The composition model enables DRY route definitions with shared prefixes.
