---
tags:
  - csharp
  - asp-net-core
  - routing
  - fundamentals
---


Understanding ASP.NET Core routing requires knowing where it came from. Historically, two distinct systems coexisted.

### Conventional Routing (MVC Legacy)

Conventional routing (also called **convention-based routing**) defines routes in a centralized location, typically in `Program.cs` (or the old `Startup.cs`). Routes are defined as templates that map URL segments to controller/action names by convention:

```csharp
// Conventional route definition
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

This single route handles URLs like:
| URL | Controller | Action | id |
|---|---|---|---|
| `/` | `Home` | `Index` | `null` |
| `/Products` | `Products` | `Index` | `null` |
| `/Products/Details/5` | `Products` | `Details` | `5` |

**Characteristics:**
- Routes are defined *away* from the controllers
- Relies on naming conventions (`{controller}` and `{action}` tokens)
- Good for MVC applications with predictable URL structures
- Hard to see which URLs a controller responds to without checking the route table

### Attribute Routing

**[[Attribute Routing]]** defines routes directly on controllers and actions using attributes like `[Route]`, `[HttpGet]`, and `[HttpPost]`:

```csharp
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetProduct(int id) => Ok();
}
```

**Characteristics:**
- Routes live *next to* the code they describe
- Explicit -- no convention magic
- Required for Web API controllers (the `[ApiController]` attribute mandates it)
- Each action clearly declares its own URL pattern

### The Tension

Before .NET Core 3.0, these two systems had separate internal implementations. Routing happened *inside* MVC, which meant middleware running before MVC had no way to know which endpoint would be selected. This created real problems for cross-cutting concerns like authorization and CORS, which needed to know the target endpoint to make decisions.

> [!warning] Common Misconception
> Conventional routing and attribute routing are **not** mutually exclusive. You can use both in the same application. Controllers without `[Route]` attributes use conventional routes; controllers with them use attribute routes. However, a single action cannot participate in both systems simultaneously.

> [!summary] Section Summary
> - Conventional routing defines routes centrally using `{controller}/{action}` templates.
> - Attribute routing defines routes on controllers and actions via attributes.
> - Before .NET Core 3.0, both systems had separate internal implementations.
> - Middleware before MVC could not inspect which endpoint was selected -- a major limitation.
> - Both systems can coexist in the same application.
