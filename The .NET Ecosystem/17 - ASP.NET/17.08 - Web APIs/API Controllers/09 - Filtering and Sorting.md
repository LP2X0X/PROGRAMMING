---
tags:
  - csharp
  - asp-net-core
  - web-api
  - controllers
---


Production APIs need more than just pagination. Clients need to **filter** data by criteria and **sort** results by specific fields.

### Filtering with Query Parameters

Define a filter model that combines pagination with filtering:

```csharp
public class ProductFilterParams : PaginationParams
{
    public string? Name { get; set; }
    public int? CategoryId { get; set; }
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
    public bool? InStock { get; set; }
}
```

Apply filters conditionally:

```csharp
[HttpGet]
public ActionResult<PagedResult<ProductDto>> GetAll(
    [FromQuery] ProductFilterParams filter)
{
    var query = _context.Products
        .Include(p => p.Category)
        .AsNoTracking();

    // Apply filters conditionally
    if (!string.IsNullOrWhiteSpace(filter.Name))
        query = query.Where(p => p.Name.Contains(filter.Name));

    if (filter.CategoryId.HasValue)
        query = query.Where(p => p.CategoryId == filter.CategoryId.Value);

    if (filter.MinPrice.HasValue)
        query = query.Where(p => p.Price >= filter.MinPrice.Value);

    if (filter.MaxPrice.HasValue)
        query = query.Where(p => p.Price <= filter.MaxPrice.Value);

    if (filter.InStock.HasValue)
        query = query.Where(p => p.StockQuantity > 0 == filter.InStock.Value);

    var result = query
        .OrderBy(p => p.Name)
        .Select(p => new ProductDto
        {
            Id = p.Id,
            Name = p.Name,
            Price = p.Price,
            CategoryName = p.Category.Name
        })
        .ToPagedResult(filter.Page, filter.PageSize);

    return Ok(result);
}
```

The request:

```http
GET /api/products?name=key&categoryId=3&minPrice=10&maxPrice=100&page=1&pageSize=20
```

### Sorting with Query Parameters

Allow clients to specify sort field and direction:

```csharp
public class ProductFilterParams : PaginationParams
{
    // ... filter properties ...

    public string? SortBy { get; set; }          // Field name: "name", "price", "date"
    public string SortOrder { get; set; } = "asc"; // "asc" or "desc"
}
```

Implement sorting with a safe whitelist approach:

```csharp
private static IQueryable<Product> ApplySorting(
    IQueryable<Product> query,
    string? sortBy,
    string sortOrder)
{
    var isDescending = sortOrder.Equals("desc", StringComparison.OrdinalIgnoreCase);

    return sortBy?.ToLower() switch
    {
        "name"     => isDescending ? query.OrderByDescending(p => p.Name)
                                   : query.OrderBy(p => p.Name),
        "price"    => isDescending ? query.OrderByDescending(p => p.Price)
                                   : query.OrderBy(p => p.Price),
        "date"     => isDescending ? query.OrderByDescending(p => p.CreatedAt)
                                   : query.OrderBy(p => p.CreatedAt),
        "category" => isDescending ? query.OrderByDescending(p => p.Category.Name)
                                   : query.OrderBy(p => p.Category.Name),
        _          => query.OrderBy(p => p.Id)  // Default sort
    };
}
```

Usage:

```http
GET /api/products?sortBy=price&sortOrder=desc&page=1&pageSize=20
```

> [!danger]
> Never pass user-provided sort field names directly into `OrderBy()` expressions using reflection or dynamic LINQ without validation. This opens the door to information disclosure or injection attacks. Always use a whitelist of allowed sort fields, as shown above.

### Search Endpoint

For more complex searching, create a dedicated search action:

```csharp
[HttpGet("search")]
public ActionResult<PagedResult<ProductDto>> Search(
    [FromQuery] string q,
    [FromQuery] PaginationParams pagination)
{
    if (string.IsNullOrWhiteSpace(q))
        return BadRequest(new { Message = "Search query 'q' is required." });

    var searchTerm = q.Trim().ToLower();

    var result = _context.Products
        .AsNoTracking()
        .Where(p => p.Name.ToLower().Contains(searchTerm)
                  || p.Description.ToLower().Contains(searchTerm)
                  || p.Category.Name.ToLower().Contains(searchTerm))
        .OrderBy(p => p.Name)
        .Select(p => p.ToDto())
        .ToPagedResult(pagination.Page, pagination.PageSize);

    return Ok(result);
}
```

```http
GET /api/products/search?q=wireless keyboard&page=1&pageSize=10
```

> [!summary] Section Summary
> Implement filtering by accepting nullable query parameters and building queries conditionally. Implement sorting with a whitelist of allowed fields to prevent injection. Combine filtering, sorting, and pagination into a single query parameter model for a flexible API.
