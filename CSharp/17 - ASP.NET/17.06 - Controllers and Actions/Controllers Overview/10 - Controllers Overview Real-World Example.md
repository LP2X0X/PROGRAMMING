---
tags:
  - csharp
  - asp-net-core
  - controllers
  - mvc
---


A complete `ProductsController` for an e-commerce API demonstrating all the concepts covered above.

### The DTO Classes

```csharp
public record ProductDto(
    int Id,
    string Name,
    string Description,
    decimal Price,
    int StockQuantity,
    string Category);

public record ProductCreateDto(
    [Required] [StringLength(200)] string Name,
    [StringLength(2000)] string Description,
    [Range(0.01, 100_000)] decimal Price,
    [Range(0, int.MaxValue)] int StockQuantity,
    [Required] string Category);

public record ProductUpdateDto(
    [Required] [StringLength(200)] string Name,
    [StringLength(2000)] string Description,
    [Range(0.01, 100_000)] decimal Price,
    [Range(0, int.MaxValue)] int StockQuantity,
    [Required] string Category);
```

### The Controller

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
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

    /// <summary>
    /// Returns all products, optionally filtered by category.
    /// </summary>
    [HttpGet]
    [ProducesResponseType(typeof(IEnumerable<ProductDto>), StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<ProductDto>>> GetAll(
        [FromQuery] string? category,
        CancellationToken cancellationToken)
    {
        _logger.LogInformation("Fetching products. Category filter: {Category}", category);

        var products = category is not null
            ? await _productRepo.GetByCategoryAsync(category, cancellationToken)
            : await _productRepo.GetAllAsync(cancellationToken);

        return Ok(products);
    }

    /// <summary>
    /// Returns a single product by its ID.
    /// </summary>
    [HttpGet("{id:int}")]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductDto>> GetById(
        int id,
        CancellationToken cancellationToken)
    {
        var product = await _productRepo.GetByIdAsync(id, cancellationToken);

        if (product is null)
        {
            _logger.LogWarning("Product {ProductId} not found", id);
            return NotFound();
        }

        return Ok(product);
    }

    /// <summary>
    /// Creates a new product.
    /// </summary>
    [HttpPost]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProductDto>> Create(
        ProductCreateDto dto,
        CancellationToken cancellationToken)
    {
        // [ApiController] guarantees ModelState is valid if we reach here

        _logger.LogInformation("Creating product: {ProductName}", dto.Name);

        var product = await _productRepo.CreateAsync(dto, cancellationToken);

        return CreatedAtAction(
            actionName: nameof(GetById),
            routeValues: new { id = product.Id },
            value: product);
    }

    /// <summary>
    /// Updates an existing product.
    /// </summary>
    [HttpPut("{id:int}")]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProductDto>> Update(
        int id,
        ProductUpdateDto dto,
        CancellationToken cancellationToken)
    {
        var existing = await _productRepo.GetByIdAsync(id, cancellationToken);

        if (existing is null)
        {
            _logger.LogWarning("Cannot update -- product {ProductId} not found", id);
            return NotFound();
        }

        _logger.LogInformation("Updating product {ProductId}", id);

        var updated = await _productRepo.UpdateAsync(id, dto, cancellationToken);
        return Ok(updated);
    }

    /// <summary>
    /// Deletes a product by its ID.
    /// </summary>
    [HttpDelete("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> Delete(
        int id,
        CancellationToken cancellationToken)
    {
        var existing = await _productRepo.GetByIdAsync(id, cancellationToken);

        if (existing is null)
        {
            _logger.LogWarning("Cannot delete -- product {ProductId} not found", id);
            return NotFound();
        }

        _logger.LogInformation("Deleting product {ProductId}", id);

        await _productRepo.DeleteAsync(id, cancellationToken);
        return NoContent();
    }
}
```

```ad-summary
**Key patterns demonstrated in the example above:**
- `ControllerBase` (not `Controller`) because this is a pure API
- `[ApiController]` for automatic validation, binding inference, and ProblemDetails
- `[Route("api/[controller]")]` for attribute-based routing
- Constructor injection for `IProductRepository` and `ILogger<T>`
- All actions are `async` with `CancellationToken`
- Strongly-typed `ActionResult<T>` for better OpenAPI documentation
- `[ProducesResponseType]` attributes for Swagger
- Proper status codes: 200, 201, 204, 400, 404
- `CreatedAtAction` returns 201 with a `Location` header pointing to the new resource
- Structured logging with message templates (not string interpolation)
```
