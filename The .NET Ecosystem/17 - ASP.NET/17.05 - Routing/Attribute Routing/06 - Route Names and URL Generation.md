---
tags:
  - csharp
  - asp-net-core
  - routing
  - attribute-routing
---


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
