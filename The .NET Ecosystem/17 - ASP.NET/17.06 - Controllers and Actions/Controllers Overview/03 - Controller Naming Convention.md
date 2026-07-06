---
tags:
  - csharp
  - asp-net-core
  - controllers
  - mvc
---


ASP.NET Core strips the `Controller` suffix from the class name when generating route tokens.

| Class Name | Route Token `[controller]` | Default URL |
|---|---|---|
| `ProductsController` | `Products` | `/Products` or `/api/Products` |
| `HomeController` | `Home` | `/Home` |
| `OrderItemsController` | `OrderItems` | `/OrderItems` |

### Convention-Based Routing (MVC)

Used primarily for MVC apps that serve views:

```csharp
// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

This maps `GET /Products/Details/5` to `ProductsController.Details(5)`.

### Attribute Routing (APIs)

Preferred for APIs. The `[controller]` token is replaced with the class name minus the `Controller` suffix:

```csharp
[ApiController]
[Route("api/[controller]")]  // resolves to "api/Products"
public class ProductsController : ControllerBase
{
    [HttpGet]           // GET api/Products
    public IActionResult GetAll() { ... }

    [HttpGet("{id}")]   // GET api/Products/5
    public IActionResult GetById(int id) { ... }

    [HttpPost]          // POST api/Products
    public IActionResult Create(ProductCreateDto dto) { ... }
}
```

See [[17.05 - Routing]] for a deep dive on routing.
