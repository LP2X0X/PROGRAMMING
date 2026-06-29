---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


**Parameter binding** determines how values from the HTTP request are mapped to handler parameters. Minimal APIs support both implicit (convention-based) and explicit (attribute-based) binding.

### Binding Attributes

```csharp
app.MapGet("/products", (
    [FromQuery] string? name,           // From query string: ?name=Widget
    [FromQuery] int? page,              // From query string: ?page=2
    [FromQuery] int? pageSize,          // From query string: ?pageSize=10
    [FromHeader(Name = "X-Api-Key")] string apiKey,  // From header
    [FromServices] IProductService service) =>        // From DI
{
    return service.Search(name, page ?? 1, pageSize ?? 10);
});

app.MapPost("/products", (
    [FromBody] CreateProductDto dto,    // From JSON body
    [FromServices] IProductService service) =>
{
    return service.Create(dto);
});

app.MapPut("/products/{id}", (
    [FromRoute] int id,                 // From route parameter
    [FromBody] UpdateProductDto dto,
    IProductService service) =>
{
    return service.Update(id, dto);
});
```

### Binding Source Summary

| Attribute        | Source          | Default For                              |
|---|---|---|
| `[FromRoute]`    | Route segment  | Parameters matching route `{name}`       |
| `[FromQuery]`    | Query string   | Simple types not in route                |
| `[FromBody]`     | Request body   | Complex types (POST/PUT/PATCH)           |
| `[FromHeader]`   | HTTP header    | Never implicit; must use attribute       |
| `[FromServices]` | DI container   | Types registered in DI                   |
| `[FromForm]`     | Form data      | When `[FromForm]` is specified (.NET 8+) |

### Implicit Binding Rules

The framework follows these rules when no explicit attribute is used:

```csharp
app.MapPost("/orders/{orderId}/items", (
    int orderId,          // [FromRoute] - matches {orderId} in the route
    string? note,         // [FromQuery] - simple type not in route -> ?note=rush
    OrderItemDto item,    // [FromBody]  - complex type -> from JSON body
    IOrderService svc,    // [FromServices] - registered in DI
    CancellationToken ct  // Special type - always available
) => { /* ... */ });
```

### The `[AsParameters]` Attribute (.NET 7+)

When you have many parameters, group them into a struct or class and use `[AsParameters]`:

```csharp
public record GetProductsRequest(
    [FromQuery] string? Name,
    [FromQuery] int Page = 1,
    [FromQuery] int PageSize = 20,
    [FromQuery] string? SortBy = "name",
    [FromHeader(Name = "X-Tenant-Id")] string? TenantId = null);

app.MapGet("/products", async (
    [AsParameters] GetProductsRequest request,
    IProductService service) =>
{
    var products = await service.SearchAsync(
        request.Name, request.Page, request.PageSize, request.SortBy);
    return Results.Ok(products);
});
```

> [!tip]
> `[AsParameters]` is especially useful when an endpoint has more than 3-4 parameters. It keeps handler signatures clean and makes the parameter contract reusable.

### Custom Binding with `TryParse` and `BindAsync`

For custom types, implement `TryParse` or `BindAsync` so the framework can bind them automatically:

```csharp
// TryParse: for simple types from route/query
public record struct Coordinate(double Latitude, double Longitude)
{
    public static bool TryParse(string? value, out Coordinate result)
    {
        result = default;
        if (value is null) return false;

        var parts = value.Split(',');
        if (parts.Length != 2) return false;

        if (double.TryParse(parts[0], out var lat) &&
            double.TryParse(parts[1], out var lng))
        {
            result = new Coordinate(lat, lng);
            return true;
        }
        return false;
    }
}

// Usage: GET /nearby?location=47.6,-122.3
app.MapGet("/nearby", (Coordinate location) =>
    Results.Ok($"Lat: {location.Latitude}, Lng: {location.Longitude}"));
```

```csharp
// BindAsync: for complex binding from the full request
public record PaginationParams(int Page, int PageSize)
{
    public static ValueTask<PaginationParams?> BindAsync(
        HttpContext context, ParameterInfo parameter)
    {
        int.TryParse(context.Request.Query["page"], out var page);
        int.TryParse(context.Request.Query["pageSize"], out var pageSize);

        var result = new PaginationParams(
            Page: page > 0 ? page : 1,
            PageSize: pageSize > 0 ? Math.Min(pageSize, 100) : 20);

        return ValueTask.FromResult<PaginationParams?>(result);
    }
}

app.MapGet("/products", (PaginationParams pagination, IProductService svc) =>
    svc.GetPaged(pagination.Page, pagination.PageSize));
```

> [!summary] Section Summary
> Parameter binding in minimal APIs uses attributes like `[FromQuery]`, `[FromRoute]`, `[FromBody]`, `[FromHeader]`, and `[FromServices]`. Implicit rules handle most cases. Use `[AsParameters]` to group many parameters into a single object. Custom types can implement `TryParse` or `BindAsync` for automatic binding.
