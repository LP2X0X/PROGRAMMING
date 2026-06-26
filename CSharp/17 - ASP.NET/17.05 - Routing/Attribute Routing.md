---
title: "Attribute Routing"
date: 2026-06-18
tags: [csharp, asp-net-core, routing, attribute-routing]
aliases: [Attribute-Based Routing, Route Attributes, API Routing]
status: complete
---

# Attribute Routing

> [!ad-note] Overview
> Attribute routing defines URL routes directly on controllers and actions using C# attributes like `[Route]`, `[HttpGet]`, and `[HttpPost]`. It is the standard routing approach for Web APIs and provides explicit, self-documenting URL mappings. This note covers all attribute routing mechanics: route composition, token replacement, multiple routes, naming, ordering, URL generation, and real-world patterns.

---

## Table of Contents

- [What Is Attribute Routing](#What%20Is%20Attribute%20Routing)
- [Route Attributes](#Route%20Attributes)
- [Controller-Level and Action-Level Routes](#Controller-Level%20and%20Action-Level%20Routes)
- [Token Replacement](#Token%20Replacement)
- [Multiple Routes on a Single Action](#Multiple%20Routes%20on%20a%20Single%20Action)
- [Route Names and URL Generation](#Route%20Names%20and%20URL%20Generation)
- [Route Order](#Route%20Order)
- [Area Routing](#Area%20Routing)
- [Attribute Routing vs Conventional Routing](#Attribute%20Routing%20vs%20Conventional%20Routing)
- [Real-World RESTful API Controller](#Real-World%20RESTful%20API%20Controller)
- [Comprehensive Summary](#Comprehensive%20Summary)
- [Related Topics](#Related%20Topics)

---

## What Is Attribute Routing

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

---

## Route Attributes

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

---

## Controller-Level and Action-Level Routes

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

---

## Token Replacement

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

---

## Multiple Routes on a Single Action

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

---

## Route Names and URL Generation

**Route names** are identifiers assigned to routes, used for generating URLs programmatically. They are not part of the URL itself -- they are internal labels.

### Assigning Route Names

```csharp
[HttpGet("{id}", Name = "GetProduct")]
public IActionResult GetById(int id) => Ok();
```

### URL Generation Using Route Names

ASP.NET Core provides several mechanisms to generate URLs from route names:

#### In Controllers

```csharp
[HttpPost]
public IActionResult Create(ProductDto dto)
{
    var product = _service.Create(dto);

    // Generate URL to the GetProduct route
    var url = Url.RouteUrl("GetProduct", new { id = product.Id });

    // Or use Url.Action (matches by controller + action name)
    var url2 = Url.Action("GetById", "Products", new { id = product.Id });

    // CreatedAtRoute -- returns 201 with Location header
    return CreatedAtRoute("GetProduct", new { id = product.Id }, product);
}
```

#### Using `LinkGenerator` (Anywhere via DI)

```csharp
public class ProductService
{
    private readonly LinkGenerator _linkGenerator;

    public ProductService(LinkGenerator linkGenerator)
    {
        _linkGenerator = linkGenerator;
    }

    public string GetProductUrl(int id, HttpContext httpContext)
    {
        return _linkGenerator.GetPathByName(
            httpContext, "GetProduct", new { id });
    }
}
```

### URL Generation Methods Comparison

| Method | Where Available | Input |
|---|---|---|
| `Url.Action()` | Controllers, Views | Controller name + action name + route values |
| `Url.RouteUrl()` | Controllers, Views | Route name + route values |
| `CreatedAtAction()` | Controllers | Action name + route values (returns 201) |
| `CreatedAtRoute()` | Controllers | Route name + route values (returns 201) |
| `LinkGenerator.GetPathByName()` | Anywhere (via DI) | Route name + route values |
| `LinkGenerator.GetPathByAction()` | Anywhere (via DI) | Action + controller + route values |

> [!tip] Practical Tip
> For Web APIs, the `CreatedAtRoute` / `CreatedAtAction` pattern is the standard way to return a 201 Created response with a `Location` header pointing to the newly created resource. This is a REST best practice.

> [!danger] Critical Warning
> Route names must be **globally unique** across the entire application. Two routes with the same name will cause a runtime exception. Use a clear naming convention like `"Get{Resource}"`, `"Create{Resource}"`, etc.

> [!summary] Section Summary
> - Route names are internal labels used for URL generation, not part of the URL.
> - `Url.Action()`, `Url.RouteUrl()`, and `LinkGenerator` generate URLs from route data.
> - `CreatedAtRoute()` and `CreatedAtAction()` return 201 responses with Location headers.
> - Route names must be globally unique across the application.
> - `LinkGenerator` can be injected anywhere, not just in controllers.

---

## Route Order

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

---

## Area Routing

**Areas** are a way to organize a large ASP.NET Core MVC application into functional groups, each with its own controllers, views, and models.

### Declaring an Area

```csharp
[Area("Admin")]
[Route("admin/[controller]")]
public class DashboardController : Controller
{
    [HttpGet]
    public IActionResult Index() => View();
    // Matches: GET admin/Dashboard
}
```

### The `[area]` Token

The `[area]` token in route templates resolves to the area name:

```csharp
[Area("Admin")]
[Route("[area]/[controller]")]
public class DashboardController : Controller
{
    [HttpGet("[action]")]
    public IActionResult Index() => View();
    // Matches: GET Admin/Dashboard/Index
}
```

### Area Routing with Conventional Routes

Areas also work with conventional routing:

```csharp
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");
```

The `{area:exists}` constraint ensures the area segment matches a registered area.

> [!summary] Section Summary
> - Areas organize large applications into functional groups with isolated controllers and views.
> - `[Area("Name")]` declares a controller's area; `[area]` token resolves to it in route templates.
> - Conventional routes support areas with the `{area:exists}` constraint.

---

## Attribute Routing vs Conventional Routing

Both approaches are valid, and understanding when to use each is important.

| Aspect | Attribute Routing | Conventional Routing |
|---|---|---|
| Route location | On controllers/actions | Centralized in `Program.cs` |
| Explicitness | Fully explicit | Convention-based |
| URL flexibility | Complete control | Limited by `{controller}/{action}` pattern |
| API suitability | Excellent (required for `[ApiController]`) | Poor for APIs |
| MVC View suitability | Good | Good (often simpler) |
| Discoverability | Look at the controller | Look at the route table |
| Refactoring impact | Route stays with the code | Centralized routes may break if controllers rename |
| `[ApiController]` compatible | Yes (required) | No |

### Guidelines

- **Web APIs**: Always use attribute routing. The `[ApiController]` attribute requires it, and explicit routes are essential for RESTful design.
- **MVC with Views**: Either works. Conventional routing is often simpler when your URLs follow the standard `{controller}/{action}/{id?}` pattern. Attribute routing when you need custom URL structures.
- **Mixed Applications**: Use both. API controllers use attribute routing; MVC controllers use conventional routing or attribute routing as needed.

> [!ad-note] Key Insight
> In practice, most modern ASP.NET Core applications use attribute routing exclusively. Conventional routing is primarily useful for traditional MVC applications with predictable URL structures. New projects tend to favor attribute routing for its explicitness and flexibility.

> [!summary] Section Summary
> - Attribute routing is explicit and required for `[ApiController]`; conventional routing is convention-based and centralized.
> - Web APIs should always use attribute routing; MVC views can use either.
> - Most modern applications favor attribute routing for its self-documenting nature.
> - Both systems can coexist in the same application.

---

## Real-World RESTful API Controller

Here is a complete, production-style API controller demonstrating the full range of attribute routing features:

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace MyApp.Controllers;

/// <summary>
/// Manages product resources.
/// Demonstrates attribute routing best practices for a RESTful API.
/// </summary>
[Route("api/[controller]")]          // Prefix: api/Products
[ApiController]                       // Requires attribute routing, enables auto-validation
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;

    public ProductsController(IProductService service)
    {
        _service = service;
    }

    /// GET api/Products
    /// Returns all products with optional filtering.
    [HttpGet]
    public async Task<ActionResult<IEnumerable<ProductDto>>> GetAll(
        [FromQuery] string? category,
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 20)
    {
        var products = await _service.GetAllAsync(category, page, pageSize);
        return Ok(products);
    }

    /// GET api/Products/5
    /// Returns a single product by ID.
    [HttpGet("{id:int}", Name = "GetProduct")]
    public async Task<ActionResult<ProductDto>> GetById(int id)
    {
        var product = await _service.GetByIdAsync(id);
        if (product is null) return NotFound();
        return Ok(product);
    }

    /// GET api/Products/by-slug/wireless-mouse
    /// Returns a product by its URL slug.
    [HttpGet("by-slug/{slug:regex(^[[a-z0-9-]]+$)}")]
    public async Task<ActionResult<ProductDto>> GetBySlug(string slug)
    {
        var product = await _service.GetBySlugAsync(slug);
        if (product is null) return NotFound();
        return Ok(product);
    }

    /// GET api/Products/5/reviews
    /// Returns reviews for a specific product (nested resource).
    [HttpGet("{productId:int}/reviews")]
    public async Task<ActionResult<IEnumerable<ReviewDto>>> GetReviews(int productId)
    {
        var reviews = await _service.GetReviewsAsync(productId);
        return Ok(reviews);
    }

    /// POST api/Products
    /// Creates a new product.
    [HttpPost]
    [Authorize(Roles = "Admin")]
    public async Task<ActionResult<ProductDto>> Create(CreateProductDto dto)
    {
        var product = await _service.CreateAsync(dto);

        // Returns 201 with Location: api/Products/{id}
        return CreatedAtRoute("GetProduct", new { id = product.Id }, product);
    }

    /// PUT api/Products/5
    /// Full update of an existing product.
    [HttpPut("{id:int}")]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> Update(int id, UpdateProductDto dto)
    {
        var exists = await _service.ExistsAsync(id);
        if (!exists) return NotFound();

        await _service.UpdateAsync(id, dto);
        return NoContent();  // 204
    }

    /// PATCH api/Products/5
    /// Partial update of an existing product.
    [HttpPatch("{id:int}")]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> PartialUpdate(
        int id,
        [FromBody] JsonPatchDocument<UpdateProductDto> patchDoc)
    {
        var product = await _service.GetByIdAsync(id);
        if (product is null) return NotFound();

        // Apply and validate the patch
        var dto = new UpdateProductDto(); // map from product
        patchDoc.ApplyTo(dto, ModelState);
        if (!ModelState.IsValid) return BadRequest(ModelState);

        await _service.UpdateAsync(id, dto);
        return NoContent();
    }

    /// DELETE api/Products/5
    /// Deletes a product.
    [HttpDelete("{id:int}")]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> Delete(int id)
    {
        var exists = await _service.ExistsAsync(id);
        if (!exists) return NotFound();

        await _service.DeleteAsync(id);
        return NoContent();  // 204
    }
}
```

### What This Controller Demonstrates

| Feature | Where Used |
|---|---|
| Controller prefix with `[controller]` token | `[Route("api/[controller]")]` |
| HTTP method attributes with templates | `[HttpGet("{id:int}")]`, `[HttpPost]`, etc. |
| Route constraints | `{id:int}`, `{slug:regex(...)}` |
| Named routes | `Name = "GetProduct"` |
| `CreatedAtRoute` for 201 responses | `Create` action |
| Nested resources | `{productId:int}/reviews` |
| Authorization on specific actions | `[Authorize(Roles = "Admin")]` |
| Query string parameters with `[FromQuery]` | `GetAll` action |

> [!summary] Section Summary
> - A well-designed RESTful controller uses `[Route("api/[controller]")]` as a prefix.
> - Each CRUD operation maps to an HTTP method: GET (read), POST (create), PUT (full update), PATCH (partial), DELETE.
> - Named routes enable `CreatedAtRoute` for proper 201 responses.
> - Route constraints (`{id:int}`) prevent invalid values from reaching action code.
> - Nested resources use compound templates like `{productId:int}/reviews`.

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Attribute routing** is the mechanism of declaring URL routes directly on controller classes and action methods using attributes like `[Route]`, `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]`, and `[HttpPatch]`. It is **required** for controllers decorated with `[ApiController]` and is the standard approach for Web APIs.
>
> Routes follow a **composition model**: the controller-level `[Route]` sets a prefix, and action-level templates are appended. Action templates beginning with `/` override the controller prefix. **Token replacement** (`[controller]`, `[action]`, `[area]`) dynamically inserts names resolved at startup, keeping routes synchronized with code.
>
> Multiple route attributes can be stacked on a single action or controller, producing a Cartesian product of all combinations. **Route names** (e.g., `Name = "GetProduct"`) enable URL generation via `Url.Action()`, `Url.RouteUrl()`, `CreatedAtRoute()`, and `LinkGenerator`. Names must be globally unique.
>
> The `Order` property controls evaluation priority but is rarely needed thanks to the built-in specificity algorithm. **Areas** add organizational grouping, supported via `[Area]` and the `[area]` token. Compared to [[Routing Overview|conventional routing]], attribute routing is more explicit and self-documenting, making it the preferred choice for most modern ASP.NET Core applications. For route parameter validation at the routing level (before the action runs), see [[Route Constraints and Tokens]].

---

## Related Topics

- [[Routing Overview]]
- [[Endpoint Routing]]
- [[Route Constraints and Tokens]]
- [[Model Binding]]
- [[Web API Design]]
- [[Controllers and Actions]]
- [[URL Generation]]

---

## Further Reading

- [[Minimal APIs]] -- an alternative to controllers that also uses attribute-style routing
- [[API Versioning]] -- how to version attribute-routed APIs
- [[Content Negotiation]] -- how `[Produces]` and `[Consumes]` work with routing
- [[Authentication and Authorization]] -- how `[Authorize]` interacts with routed endpoints
