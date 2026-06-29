---
tags:
  - csharp
  - asp-net-core
  - web-api
  - controllers
---


Production APIs must paginate list endpoints. Returning thousands of records in a single response wastes bandwidth, memory, and time. ==Always paginate collection endpoints.==

### Query Parameters for Pagination

The standard approach uses `page` (or `pageNumber`) and `pageSize` query parameters:

```http
GET /api/products?page=2&pageSize=25
```

### Pagination Models

```csharp
// Request model for pagination parameters
public class PaginationParams
{
    private const int MaxPageSize = 100;
    private int _pageSize = 10;

    public int Page { get; set; } = 1;
    
    public int PageSize
    {
        get => _pageSize;
        set => _pageSize = value > MaxPageSize ? MaxPageSize : value;
    }
}

// Response wrapper with pagination metadata
public class PagedResult<T>
{
    public IEnumerable<T> Items { get; set; } = Enumerable.Empty<T>();
    public int Page { get; set; }
    public int PageSize { get; set; }
    public int TotalCount { get; set; }
    public int TotalPages => (int)Math.Ceiling(TotalCount / (double)PageSize);
    public bool HasPreviousPage => Page > 1;
    public bool HasNextPage => Page < TotalPages;
}
```

### Implementing Pagination

```csharp
[HttpGet]
[ProducesResponseType(typeof(PagedResult<ProductDto>), StatusCodes.Status200OK)]
public ActionResult<PagedResult<ProductDto>> GetAll(
    [FromQuery] PaginationParams paginationParams)
{
    var query = _context.Products
        .Include(p => p.Category)
        .AsNoTracking();

    var totalCount = query.Count();

    var products = query
        .OrderBy(p => p.Id)
        .Skip((paginationParams.Page - 1) * paginationParams.PageSize)
        .Take(paginationParams.PageSize)
        .Select(p => new ProductDto
        {
            Id = p.Id,
            Name = p.Name,
            Description = p.Description,
            Price = p.Price,
            CategoryName = p.Category.Name
        })
        .ToList();

    var result = new PagedResult<ProductDto>
    {
        Items = products,
        Page = paginationParams.Page,
        PageSize = paginationParams.PageSize,
        TotalCount = totalCount
    };

    return Ok(result);
}
```

The response looks like:

```json
{
    "items": [
        { "id": 26, "name": "USB Hub", "price": 24.99, "categoryName": "Accessories" },
        { "id": 27, "name": "Webcam", "price": 79.99, "categoryName": "Peripherals" }
    ],
    "page": 2,
    "pageSize": 25,
    "totalCount": 142,
    "totalPages": 6,
    "hasPreviousPage": true,
    "hasNextPage": true
}
```

### Adding Pagination Headers

Some APIs return pagination metadata in response headers instead of (or in addition to) the body:

```csharp
[HttpGet]
public ActionResult<IEnumerable<ProductDto>> GetAll(
    [FromQuery] PaginationParams paginationParams)
{
    var query = _context.Products.AsNoTracking();
    var totalCount = query.Count();
    var totalPages = (int)Math.Ceiling(totalCount / (double)paginationParams.PageSize);

    Response.Headers.Append("X-Total-Count", totalCount.ToString());
    Response.Headers.Append("X-Total-Pages", totalPages.ToString());
    Response.Headers.Append("X-Page", paginationParams.Page.ToString());
    Response.Headers.Append("X-Page-Size", paginationParams.PageSize.ToString());

    var products = query
        .OrderBy(p => p.Id)
        .Skip((paginationParams.Page - 1) * paginationParams.PageSize)
        .Take(paginationParams.PageSize)
        .Select(p => p.ToDto())
        .ToList();

    return Ok(products);
}
```

> [!tip]
> If you use custom headers, remember to expose them in your CORS policy:
> ```csharp
> builder.Services.AddCors(options =>
> {
>     options.AddDefaultPolicy(policy =>
>     {
>         policy.WithExposedHeaders(
>             "X-Total-Count", "X-Total-Pages", "X-Page", "X-Page-Size");
>     });
> });
> ```

### Reusable Pagination Extension Method

Create an extension method to apply pagination to any `IQueryable<T>`:

```csharp
public static class QueryableExtensions
{
    public static PagedResult<T> ToPagedResult<T>(
        this IQueryable<T> query, 
        int page, 
        int pageSize)
    {
        var totalCount = query.Count();

        var items = query
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .ToList();

        return new PagedResult<T>
        {
            Items = items,
            Page = page,
            PageSize = pageSize,
            TotalCount = totalCount
        };
    }
}
```

Usage:

```csharp
[HttpGet]
public ActionResult<PagedResult<ProductDto>> GetAll([FromQuery] PaginationParams p)
{
    var result = _context.Products
        .AsNoTracking()
        .OrderBy(x => x.Id)
        .Select(x => x.ToDto())
        .ToPagedResult(p.Page, p.PageSize);

    return Ok(result);
}
```

> [!summary] Section Summary
> Always paginate collection endpoints using `page` and `pageSize` query parameters. Cap the maximum page size to prevent abuse. Return pagination metadata (total count, total pages, has next/previous) either in the response body or via custom headers. Use reusable extension methods to keep pagination logic DRY.
