---
tags:
  - csharp
  - asp-net-core
  - controllers
  - mvc
---


ASP.NET Core provides two base classes for controllers. Choosing the right one depends on whether you are building an **API** or an **MVC app with views**.

### Class Hierarchy

```
Microsoft.AspNetCore.Mvc.ControllerBase
    |
    +-- Microsoft.AspNetCore.Mvc.Controller
```

`Controller` inherits from `ControllerBase` and adds view-rendering capabilities on top.

### ControllerBase (for APIs)

`ControllerBase` is the lean base class designed for API controllers. It provides everything you need to handle HTTP requests and produce responses without any view-rendering overhead.

**Key Properties:**

| Property      | Description                                               |
|---|---|
| `HttpContext` | The full HTTP context for the current request             |
| `Request`     | The `HttpRequest` object (headers, query, body, etc.)     |
| `Response`    | The `HttpResponse` object (status code, headers, cookies) |
| `ModelState`  | Validation state of the model-bound data                  | 
| `User`        | The `ClaimsPrincipal` representing the authenticated user |
| `Url`         | URL generation helper                                     |

**Key Methods:**

| Method           | Status Code | Usage                                  |
|---|---|---|
| `Ok()`           | 200         | Successful response with optional body |
| `Created()`      | 201         | Resource successfully created          |
| `NoContent()`    | 204         | Success with no response body          |
| `BadRequest()`   | 400         | Client sent invalid data               |
| `Unauthorized()` | 401         | Authentication required                |
| `Forbid()`       | 403         | Authenticated but not authorized       |
| `NotFound()`     | 404         | Resource does not exist                | 
| `Conflict()`     | 409         | Request conflicts with current state   |
| `StatusCode()`   | Any         | Return any arbitrary status code       |

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var order = _orderService.Find(id);
        if (order is null)
            return NotFound();

        return Ok(order);
    }
}
```

### Controller (for MVC with Views)

`Controller` inherits everything from `ControllerBase` and adds view-rendering support. Use this when your actions return HTML pages via Razor views (see [[17.07 - Views and Razor]]).

**Additional Properties:**

| Property | Description |
|---|---|
| `ViewData` | Dictionary for passing data to views |
| `ViewBag` | Dynamic wrapper around `ViewData` |
| `TempData` | Data that survives a single redirect |

**Additional Methods:**

| Method | Description |
|---|---|
| `View()` | Renders a Razor view |
| `PartialView()` | Renders a partial view |
| `Json()` | Serializes an object to JSON |
| `RedirectToAction()` | Redirects to another action |

```csharp
public class ProductsController : Controller
{
    [HttpGet]
    public IActionResult Index()
    {
        var products = _productService.GetAll();
        ViewBag.Title = "All Products";
        return View(products); // renders Views/Products/Index.cshtml
    }

    [HttpGet]
    public IActionResult Details(int id)
    {
        var product = _productService.Find(id);
        if (product is null)
            return NotFound();

        return View(product);
    }
}
```

```ad-tip
**Guideline:** Use `ControllerBase` for API controllers and `Controller` for MVC apps that render views. If you are building a pure API (which is the most common modern scenario), there is no reason to inherit from `Controller` and carry the view-rendering baggage.
```
