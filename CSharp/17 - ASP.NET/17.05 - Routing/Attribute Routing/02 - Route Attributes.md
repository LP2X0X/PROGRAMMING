---
tags:
  - csharp
  - asp-net-core
  - routing
  - attribute-routing
---


ASP.NET Core provides several attributes for defining routes, each tied to a specific HTTP method.

### The `[Route]` Attribute

The general-purpose route attribute. It defines a URL pattern but does **not** constrain the HTTP method:

```csharp
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [Route("featured")]    // Matches ALL HTTP methods at api/products/featured
    public IActionResult Featured() => Ok();
}
```

### HTTP Method Attributes

These combine a route template with an HTTP method constraint:

| Attribute | HTTP Method | Example |
|---|---|---|
| `[HttpGet]` | GET | `[HttpGet("items")]` |
| `[HttpPost]` | POST | `[HttpPost]` |
| `[HttpPut]` | PUT | `[HttpPut("{id}")]` |
| `[HttpDelete]` | DELETE | `[HttpDelete("{id}")]` |
| `[HttpPatch]` | PATCH | `[HttpPatch("{id}")]` |
| `[HttpHead]` | HEAD | `[HttpHead]` |
| `[HttpOptions]` | OPTIONS | `[HttpOptions]` |

> [!tip] Practical Tip
> Prefer `[HttpGet]`, `[HttpPost]`, etc. over `[Route]` for actions. The HTTP method attributes are more expressive and constrain which methods are allowed. Use `[Route]` primarily on controllers to set a prefix, or when an action genuinely needs to respond to any HTTP method.

### Attribute Without a Template

When an HTTP method attribute is used without a template, the action matches the controller-level route directly:

```csharp
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [HttpGet]     // GET api/products  (no additional path segment)
    public IActionResult GetAll() => Ok();

    [HttpPost]    // POST api/products
    public IActionResult Create() => Ok();
}
```

> [!summary] Section Summary
> - `[Route]` defines a URL pattern without constraining the HTTP method.
> - `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]`, and `[HttpPatch]` combine a template with a method constraint.
> - HTTP method attributes without a template match the controller-level route path.
> - Prefer HTTP method attributes over `[Route]` for action methods.
