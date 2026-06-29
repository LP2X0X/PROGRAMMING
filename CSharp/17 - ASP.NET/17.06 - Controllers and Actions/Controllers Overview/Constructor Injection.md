---
tags:
  - csharp
  - asp-net-core
  - controllers
  - mvc
---


Controllers receive their dependencies through constructor injection from the built-in DI container. This is the standard way to get services into a controller.

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductRepository _productRepo;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(
        IProductRepository productRepo,
        ILogger<ProductsController> logger)
    {
        _productRepo = productRepo;
        _logger = logger;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id)
    {
        _logger.LogInformation("Fetching product {ProductId}", id);
        var product = await _productRepo.GetByIdAsync(id);

        if (product is null)
        {
            _logger.LogWarning("Product {ProductId} not found", id);
            return NotFound();
        }

        return Ok(product);
    }
}
```

```ad-info
Controllers have a **transient** lifetime -- a new instance is created for every HTTP request. This means constructor parameters are resolved fresh each time, which is safe even for scoped services like `DbContext`.
```

You can also inject services into individual actions using the `[FromServices]` attribute, which is useful for dependencies needed by only one action:

```csharp
[HttpPost("export")]
public async Task<IActionResult> Export(
    [FromServices] IExportService exportService)
{
    var data = await exportService.GenerateReportAsync();
    return File(data, "application/pdf", "report.pdf");
}
```

See [[17.03 - Dependency Injection]] for full coverage of the DI container, lifetimes, and registration patterns.
