---
tags:
  - csharp
  - asp-net-core
  - web-api
  - controllers
---


Here is a production-quality `ProductsController` combining all concepts: CRUD operations, DTOs, pagination, filtering, sorting, proper status codes, and OpenAPI documentation.

### The Entity Model

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public decimal CostPrice { get; set; }
    public int StockQuantity { get; set; }
    public bool IsActive { get; set; } = true;
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
    
    public int CategoryId { get; set; }
    public Category Category { get; set; } = null!;
}
```

### The DTOs

```csharp
// Output DTO
public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public bool InStock => StockQuantity > 0;
    public string CategoryName { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; }
}

// Create input DTO
public class CreateProductDto
{
    [Required(ErrorMessage = "Product name is required.")]
    [StringLength(200, MinimumLength = 1, ErrorMessage = "Name must be between 1 and 200 characters.")]
    public string Name { get; set; } = string.Empty;

    [StringLength(2000, ErrorMessage = "Description cannot exceed 2000 characters.")]
    public string Description { get; set; } = string.Empty;

    [Required(ErrorMessage = "Price is required.")]
    [Range(0.01, 99999.99, ErrorMessage = "Price must be between 0.01 and 99999.99.")]
    public decimal Price { get; set; }

    [Range(0, int.MaxValue, ErrorMessage = "Stock quantity cannot be negative.")]
    public int StockQuantity { get; set; }

    [Required(ErrorMessage = "Category is required.")]
    public int CategoryId { get; set; }
}

// Update input DTO
public class UpdateProductDto
{
    [Required(ErrorMessage = "Product name is required.")]
    [StringLength(200, MinimumLength = 1)]
    public string Name { get; set; } = string.Empty;

    [StringLength(2000)]
    public string Description { get; set; } = string.Empty;

    [Required]
    [Range(0.01, 99999.99)]
    public decimal Price { get; set; }

    [Range(0, int.MaxValue)]
    public int StockQuantity { get; set; }

    [Required]
    public int CategoryId { get; set; }
}
```

### The Filter/Pagination Model

```csharp
public class ProductQueryParams
{
    private const int MaxPageSize = 50;
    private int _pageSize = 10;

    // Pagination
    public int Page { get; set; } = 1;
    public int PageSize
    {
        get => _pageSize;
        set => _pageSize = value > MaxPageSize ? MaxPageSize : (value < 1 ? 1 : value);
    }

    // Filtering
    public string? Search { get; set; }
    public int? CategoryId { get; set; }
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
    public bool? InStock { get; set; }

    // Sorting
    public string? SortBy { get; set; }
    public string SortOrder { get; set; } = "asc";
}
```

### The Complete Controller

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    private readonly AppDbContext _context;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(AppDbContext context, ILogger<ProductsController> logger)
    {
        _context = context;
        _logger = logger;
    }

    /// <summary>
    /// Gets a paginated, filterable, sortable list of products.
    /// </summary>
    [HttpGet]
    [ProducesResponseType(typeof(PagedResult<ProductDto>), StatusCodes.Status200OK)]
    public ActionResult<PagedResult<ProductDto>> GetAll(
        [FromQuery] ProductQueryParams queryParams)
    {
        var query = _context.Products
            .Include(p => p.Category)
            .Where(p => p.IsActive)
            .AsNoTracking();

        // Apply filters
        if (!string.IsNullOrWhiteSpace(queryParams.Search))
        {
            var search = queryParams.Search.Trim().ToLower();
            query = query.Where(p =>
                p.Name.ToLower().Contains(search) ||
                p.Description.ToLower().Contains(search));
        }

        if (queryParams.CategoryId.HasValue)
            query = query.Where(p => p.CategoryId == queryParams.CategoryId.Value);

        if (queryParams.MinPrice.HasValue)
            query = query.Where(p => p.Price >= queryParams.MinPrice.Value);

        if (queryParams.MaxPrice.HasValue)
            query = query.Where(p => p.Price <= queryParams.MaxPrice.Value);

        if (queryParams.InStock.HasValue)
            query = query.Where(p => (p.StockQuantity > 0) == queryParams.InStock.Value);

        // Apply sorting
        query = ApplySorting(query, queryParams.SortBy, queryParams.SortOrder);

        // Apply pagination
        var totalCount = query.Count();
        var products = query
            .Skip((queryParams.Page - 1) * queryParams.PageSize)
            .Take(queryParams.PageSize)
            .Select(p => MapToDto(p))
            .ToList();

        var result = new PagedResult<ProductDto>
        {
            Items = products,
            Page = queryParams.Page,
            PageSize = queryParams.PageSize,
            TotalCount = totalCount
        };

        return Ok(result);
    }

    /// <summary>
    /// Gets a single product by its ID.
    /// </summary>
    [HttpGet("{id:int}", Name = "GetProductById")]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public ActionResult<ProductDto> GetById(int id)
    {
        var product = _context.Products
            .Include(p => p.Category)
            .AsNoTracking()
            .FirstOrDefault(p => p.Id == id && p.IsActive);

        if (product is null)
        {
            _logger.LogWarning("Product with ID {ProductId} not found.", id);
            return NotFound();
        }

        return MapToDto(product);
    }

    /// <summary>
    /// Creates a new product.
    /// </summary>
    [HttpPost]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public ActionResult<ProductDto> Create(CreateProductDto dto)
    {
        // Validate that the category exists
        var categoryExists = _context.Categories.Any(c => c.Id == dto.CategoryId);
        if (!categoryExists)
        {
            ModelState.AddModelError(nameof(dto.CategoryId), 
                $"Category with ID {dto.CategoryId} does not exist.");
            return ValidationProblem(ModelState);
        }

        var product = new Product
        {
            Name = dto.Name,
            Description = dto.Description,
            Price = dto.Price,
            StockQuantity = dto.StockQuantity,
            CategoryId = dto.CategoryId,
            CreatedAt = DateTime.UtcNow,
            IsActive = true
        };

        _context.Products.Add(product);
        _context.SaveChanges();

        // Reload with navigation properties for the response
        _context.Entry(product).Reference(p => p.Category).Load();

        _logger.LogInformation("Product created with ID {ProductId}.", product.Id);

        return CreatedAtRoute(
            "GetProductById",
            new { id = product.Id },
            MapToDto(product));
    }

    /// <summary>
    /// Fully updates an existing product.
    /// </summary>
    [HttpPut("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public IActionResult Update(int id, UpdateProductDto dto)
    {
        var product = _context.Products.Find(id);
        if (product is null || !product.IsActive)
            return NotFound();

        // Validate category
        var categoryExists = _context.Categories.Any(c => c.Id == dto.CategoryId);
        if (!categoryExists)
        {
            ModelState.AddModelError(nameof(dto.CategoryId),
                $"Category with ID {dto.CategoryId} does not exist.");
            return ValidationProblem(ModelState);
        }

        product.Name = dto.Name;
        product.Description = dto.Description;
        product.Price = dto.Price;
        product.StockQuantity = dto.StockQuantity;
        product.CategoryId = dto.CategoryId;
        product.UpdatedAt = DateTime.UtcNow;

        _context.SaveChanges();

        _logger.LogInformation("Product {ProductId} updated.", id);

        return NoContent();
    }

    /// <summary>
    /// Deletes a product (soft delete).
    /// </summary>
    [HttpDelete("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public IActionResult Delete(int id)
    {
        var product = _context.Products.Find(id);
        if (product is null || !product.IsActive)
            return NotFound();

        // Soft delete
        product.IsActive = false;
        product.UpdatedAt = DateTime.UtcNow;
        _context.SaveChanges();

        _logger.LogInformation("Product {ProductId} soft-deleted.", id);

        return NoContent();
    }

    // ---- Private helpers ----

    private static ProductDto MapToDto(Product product)
    {
        return new ProductDto
        {
            Id = product.Id,
            Name = product.Name,
            Description = product.Description,
            Price = product.Price,
            StockQuantity = product.StockQuantity,
            CategoryName = product.Category?.Name ?? string.Empty,
            CreatedAt = product.CreatedAt
        };
    }

    private static IQueryable<Product> ApplySorting(
        IQueryable<Product> query, string? sortBy, string sortOrder)
    {
        var descending = sortOrder.Equals("desc", StringComparison.OrdinalIgnoreCase);

        return sortBy?.ToLower() switch
        {
            "name"     => descending ? query.OrderByDescending(p => p.Name)
                                     : query.OrderBy(p => p.Name),
            "price"    => descending ? query.OrderByDescending(p => p.Price)
                                     : query.OrderBy(p => p.Price),
            "stock"    => descending ? query.OrderByDescending(p => p.StockQuantity)
                                     : query.OrderBy(p => p.StockQuantity),
            "date"     => descending ? query.OrderByDescending(p => p.CreatedAt)
                                     : query.OrderBy(p => p.CreatedAt),
            "category" => descending ? query.OrderByDescending(p => p.Category.Name)
                                     : query.OrderBy(p => p.Category.Name),
            _          => query.OrderBy(p => p.Id)
        };
    }
}
```

### Example API Calls

List products with filtering, sorting, and pagination:

```http
GET /api/products?search=wireless&categoryId=3&minPrice=20&maxPrice=100&sortBy=price&sortOrder=asc&page=1&pageSize=10
```

Get a single product:

```http
GET /api/products/42
```

Create a product:

```http
POST /api/products
Content-Type: application/json

{
    "name": "Mechanical Keyboard",
    "description": "RGB mechanical keyboard with Cherry MX switches",
    "price": 129.99,
    "stockQuantity": 50,
    "categoryId": 3
}
```

Response:

```http
HTTP/1.1 201 Created
Location: /api/products/42
Content-Type: application/json

{
    "id": 42,
    "name": "Mechanical Keyboard",
    "description": "RGB mechanical keyboard with Cherry MX switches",
    "price": 129.99,
    "stockQuantity": 50,
    "inStock": true,
    "categoryName": "Peripherals",
    "createdAt": "2026-06-18T14:30:00Z"
}
```

Update a product:

```http
PUT /api/products/42
Content-Type: application/json

{
    "name": "Mechanical Keyboard Pro",
    "description": "Updated RGB mechanical keyboard with Cherry MX Brown switches",
    "price": 149.99,
    "stockQuantity": 45,
    "categoryId": 3
}
```

Delete a product:

```http
DELETE /api/products/42
```

> [!example]
> **Testing with curl:**
> ```bash
> # List products
> curl -s https://localhost:5001/api/products?page=1&pageSize=5 | jq
>
> # Create a product
> curl -s -X POST https://localhost:5001/api/products \
>   -H "Content-Type: application/json" \
>   -d '{"name":"Monitor","description":"27 inch 4K","price":499.99,"stockQuantity":20,"categoryId":1}' | jq
>
> # Update a product
> curl -s -X PUT https://localhost:5001/api/products/1 \
>   -H "Content-Type: application/json" \
>   -d '{"name":"Monitor Pro","description":"27 inch 4K HDR","price":599.99,"stockQuantity":15,"categoryId":1}'
>
> # Delete a product
> curl -s -X DELETE https://localhost:5001/api/products/1
> ```

> [!summary] Section Summary
> A production-quality API controller combines `[ApiController]`, `ControllerBase`, attribute routing, `ActionResult<T>` return types, separate DTOs for input and output, pagination, filtering, sorting, proper status codes, logging, and OpenAPI annotations. The complete `ProductsController` above demonstrates all of these patterns working together.
