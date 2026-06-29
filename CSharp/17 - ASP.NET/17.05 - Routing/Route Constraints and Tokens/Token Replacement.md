---
tags:
  - csharp
  - asp-net-core
  - routing
  - constraints
---


Token replacement is the mechanism by which special placeholders in route templates are resolved to actual names at application startup.

### The Three Tokens

| Token | Resolves To | Example Class/Method | Result |
|---|---|---|---|
| `[controller]` | Controller class name minus "Controller" suffix | `ProductsController` | `Products` |
| `[action]` | Action method name | `GetById` | `GetById` |
| `[area]` | Area name from `[Area]` attribute | `[Area("Admin")]` | `Admin` |

### How It Works

```csharp
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpGet("[action]/{id}")]
    public IActionResult GetById(int id) => Ok();
}
// Resolved route: GET api/Orders/GetById/{id}
```

### Resolution Timing

Tokens are resolved **once at application startup**, not per-request. The resolved route template is compiled into the route table and used for all subsequent matching. This means:
- Zero runtime overhead for token resolution
- Renaming a class/method changes the route (which is the point)
- You cannot conditionally change token values at runtime

### Token vs Literal Trade-offs

| Approach | Pros | Cons |
|---|---|---|
| `[Route("api/[controller]")]` | Auto-syncs with class name; DRY | Renaming class changes public URL (breaking change) |
| `[Route("api/products")]` | URL is explicit and stable | Manual sync needed; can drift from class name |

> [!tip] Practical Tip
> For internal APIs or early-stage development, `[controller]` tokens are convenient. For **public-facing APIs** where URL stability matters, consider hardcoding the route string. A rename refactoring tool will not know to update your API consumers.

> [!summary] Section Summary
> - `[controller]`, `[action]`, and `[area]` tokens are resolved at startup, not per-request.
> - They keep routes synchronized with code names automatically.
> - For public APIs, hardcoded route strings provide URL stability.
> - Tokens have zero runtime performance cost.
