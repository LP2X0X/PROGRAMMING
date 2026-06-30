---
tags:
  - csharp
  - asp-net-core
  - routing
  - attribute-routing
---


Token replacement allows route templates to reference the controller name, action name, or area name dynamically. This keeps routes in sync with class/method names without hardcoding.

### Available Tokens

| Token | Replaced With | Example |
|---|---|---|
| `[controller]` | Controller class name minus the "Controller" suffix | `ProductsController` -> `Products` |
| `[action]` | Action method name | `GetById` -> `GetById` |
| `[area]` | Area name (if using areas) | `Admin` -> `Admin` |

### Usage

```csharp
[Route("api/[controller]")]    // Resolves to "api/Products"
public class ProductsController : ControllerBase
{
    [HttpGet("[action]")]      // Resolves to "api/Products/GetAll"
    public IActionResult GetAll() => Ok();

    [HttpGet("[action]/{id}")] // Resolves to "api/Products/GetById/5"
    public IActionResult GetById(int id) => Ok();
}
```

### Token Case Transformation

Tokens use the exact name from the class or method. `ProductsController` becomes `Products` (PascalCase). If you want **kebab-case** or **lowercase** URLs, use a **parameter transformer**:

```csharp
// In Program.cs -- configure a slugify transformer
builder.Services.AddRouting(options =>
{
    options.LowercaseUrls = true;
    options.LowercaseQueryStrings = true;
});
```

For more advanced transformations (PascalCase to kebab-case), see [[Route Constraints and Tokens]].

> [!tip] Practical Tip
> `[controller]` is extremely common on API controllers. It keeps your routes in sync with your class names automatically. If you rename `ProductsController` to `CatalogController`, the route updates to `api/Catalog` without any manual changes.

> [!warning] Common Misconception
> Tokens are replaced at **application startup**, not at request time. They are not dynamic -- they resolve once to a fixed string. This means they have zero performance cost at runtime.

> [!summary] Section Summary
> - `[controller]`, `[action]`, and `[area]` are tokens replaced at startup with actual names.
> - `[controller]` strips the "Controller" suffix from the class name.
> - Tokens keep routes in sync with code -- renaming a controller automatically changes its route.
> - For lowercase or kebab-case URLs, configure `LowercaseUrls` or use parameter transformers.
