---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - fundamentals
---

## How Controllers Get Injected Dependencies

ASP.NET Core controllers are created by the framework's controller activator, which uses the DI container to resolve all constructor dependencies automatically.

### Complete Example: Registration to Resolution

**Step 1 -- Define the interfaces and implementations:**

```csharp
// Interfaces
public interface IProductService
{
    Task<Product?> GetByIdAsync(int id);
    Task<IEnumerable<Product>> SearchAsync(string query);
    Task<Product> CreateAsync(CreateProductRequest request);
}

public interface IProductRepository
{
    Task<Product?> FindByIdAsync(int id);
    Task<IEnumerable<Product>> SearchAsync(string query);
    Task<Product> AddAsync(Product product);
}

// Implementation
public class ProductService : IProductService
{
    private readonly IProductRepository _repository;
    private readonly ILogger<ProductService> _logger;

    public ProductService(IProductRepository repository, ILogger<ProductService> logger)
    {
        _repository = repository;
        _logger = logger;
    }

    public async Task<Product?> GetByIdAsync(int id)
    {
        _logger.LogInformation("Fetching product {ProductId}", id);
        return await _repository.FindByIdAsync(id);
    }

    public async Task<IEnumerable<Product>> SearchAsync(string query)
    {
        _logger.LogInformation("Searching products with query: {Query}", query);
        return await _repository.SearchAsync(query);
    }

    public async Task<Product> CreateAsync(CreateProductRequest request)
    {
        var product = new Product
        {
            Name = request.Name,
            Price = request.Price,
            Category = request.Category
        };

        _logger.LogInformation("Creating product: {ProductName}", product.Name);
        return await _repository.AddAsync(product);
    }
}
```

**Step 2 -- Register services in Program.cs:**

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Register application services
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddScoped<IProductRepository, SqlProductRepository>();

var app = builder.Build();

app.MapControllers();
app.Run();
```

**Step 3 -- The controller receives everything automatically:**

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductController : ControllerBase
{
    private readonly IProductService _productService;
    private readonly ILogger<ProductController> _logger;

    // The DI container provides both IProductService and ILogger automatically
    public ProductController(
        IProductService productService,
        ILogger<ProductController> logger)
    {
        _productService = productService;
        _logger = logger;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetProduct(int id)
    {
        var product = await _productService.GetByIdAsync(id);
        if (product is null)
        {
            _logger.LogWarning("Product {ProductId} not found", id);
            return NotFound();
        }
        return Ok(product);
    }

    [HttpGet("search")]
    public async Task<IActionResult> Search([FromQuery] string query)
    {
        var results = await _productService.SearchAsync(query);
        return Ok(results);
    }

    [HttpPost]
    public async Task<IActionResult> CreateProduct([FromBody] CreateProductRequest request)
    {
        var product = await _productService.CreateAsync(request);
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }
}
```

> [!ad-note]
> Notice that `ILogger<T>` is never explicitly registered by you. ASP.NET Core registers logging services automatically as part of `WebApplication.CreateBuilder()`. Many framework services -- logging, configuration, hosting -- are pre-registered for you. You only need to register your own application-specific services.

### What Happens at Runtime

When a request hits `GET /api/product/42`:

1. The framework routes the request to `ProductController.GetProduct`.
2. The controller activator asks the DI container: "Create a `ProductController`."
3. The container sees the constructor needs `IProductService` and `ILogger<ProductController>`.
4. It resolves `IProductService` -> `ProductService`, which needs `IProductRepository` and `ILogger<ProductService>`.
5. It resolves `IProductRepository` -> `SqlProductRepository`.
6. It resolves both `ILogger<>` instances from the pre-registered logging services.
7. It builds the full object graph: `SqlProductRepository` -> `ProductService` -> `ProductController`.
8. The controller's `GetProduct` method executes with all dependencies in place.

> [!warning] Common Misconception
> You do not need to register controllers themselves as services for constructor injection to work. `AddControllers()` handles controller discovery and activation. Controllers are not resolved from the DI container by default -- they are created by the `DefaultControllerActivator`, which uses the container only to resolve the constructor *parameters*. If you want controllers themselves to be container-managed (for filters, etc.), you can call `builder.Services.AddControllers().AddControllersAsServices()`, but this is not required for basic DI.

> [!summary] Section Summary
> - Controllers declare their dependencies as constructor parameters; the framework resolves them automatically.
> - You register your application services in `Program.cs`; framework services like `ILogger<T>` are pre-registered.
> - The DI container builds the entire dependency graph recursively at the time of controller activation.
> - `AddControllers()` handles controller discovery -- you do not register controllers as services for basic DI.
> - The full resolution chain (controller -> service -> repository) happens transparently on every request.
