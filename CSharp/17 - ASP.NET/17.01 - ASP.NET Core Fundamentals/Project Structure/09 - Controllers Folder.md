---
tags: [csharp, asp-net-core, project-structure]
---


The `Controllers/` folder contains controller classes that handle HTTP requests in MVC and Web API projects. Each controller groups related endpoints together.

### Web API Controller Example

```csharp
using Microsoft.AspNetCore.Mvc;

namespace InventoryApi.Controllers;

[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductRepository _repository;

    public ProductsController(IProductRepository repository)
    {
        _repository = repository;
    }

    [HttpGet]
    public async Task<ActionResult<IEnumerable<Product>>> GetAll()
    {
        var products = await _repository.GetAllAsync();
        return Ok(products);
    }

    [HttpGet("{id:int}")]
    public async Task<ActionResult<Product>> GetById(int id)
    {
        var product = await _repository.GetByIdAsync(id);
        if (product is null)
            return NotFound();

        return Ok(product);
    }

    [HttpPost]
    public async Task<ActionResult<Product>> Create(CreateProductRequest request)
    {
        var product = await _repository.CreateAsync(request);
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }
}
```

### MVC Controller Example

```csharp
using Microsoft.AspNetCore.Mvc;

namespace ProductCatalog.Controllers;

public class OrdersController : Controller
{
    private readonly IOrderService _orderService;

    public OrdersController(IOrderService orderService)
    {
        _orderService = orderService;
    }

    public async Task<IActionResult> Index()
    {
        var orders = await _orderService.GetRecentOrdersAsync();
        return View(orders);
    }

    public async Task<IActionResult> Details(int id)
    {
        var order = await _orderService.GetByIdAsync(id);
        if (order is null)
            return NotFound();

        return View(order);
    }
}
```

> [!ad-note] ControllerBase vs Controller
> - `ControllerBase` -- used for Web APIs. Provides action result helpers (`Ok()`, `NotFound()`, `BadRequest()`) but no view support.
> - `Controller` -- inherits from `ControllerBase` and adds view-related features (`View()`, `ViewBag`, `ViewData`, `TempData`). Used for MVC applications.

> [!summary] Section Summary
> - `Controllers/` contains classes that handle HTTP requests and return responses
> - Web API controllers inherit from `ControllerBase`; MVC controllers inherit from `Controller`
> - `[ApiController]` and `[Route]` attributes configure routing and behavior for API controllers
> - Controllers receive dependencies through constructor injection
