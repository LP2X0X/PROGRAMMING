---
tags:
  - csharp
  - asp-net-core
  - routing
  - endpoints
---


Minimal APIs (introduced in .NET 6) define endpoints as inline delegates without controllers:

```csharp
// Simple endpoints
app.MapGet("/hello", () => "Hello World");
app.MapGet("/hello/{name}", (string name) => $"Hello {name}");

// With parameter binding and return types
app.MapGet("/products/{id:int}", async (int id, IProductService service) =>
{
    var product = await service.GetByIdAsync(id);
    return product is not null
        ? Results.Ok(product)
        : Results.NotFound();
});

app.MapPost("/products", async (CreateProductDto dto, IProductService service) =>
{
    var product = await service.CreateAsync(dto);
    return Results.Created($"/products/{product.Id}", product);
});

app.MapPut("/products/{id:int}", async (int id, UpdateProductDto dto, IProductService service) =>
{
    await service.UpdateAsync(id, dto);
    return Results.NoContent();
});

app.MapDelete("/products/{id:int}", async (int id, IProductService service) =>
{
    await service.DeleteAsync(id);
    return Results.NoContent();
});
```

### Available Map Methods for Minimal APIs

| Method | HTTP Method |
|---|---|
| `MapGet()` | GET |
| `MapPost()` | POST |
| `MapPut()` | PUT |
| `MapDelete()` | DELETE |
| `MapPatch()` | PATCH |
| `MapMethods()` | Custom set of methods |
| `Map()` | All methods |

### Chaining Metadata

Minimal API endpoints support fluent metadata chaining:

```csharp
app.MapGet("/admin/dashboard", () => "Admin Dashboard")
    .RequireAuthorization("AdminPolicy")
    .WithName("AdminDashboard")
    .WithTags("Admin")
    .Produces<DashboardDto>(StatusCodes.Status200OK)
    .ProducesProblem(StatusCodes.Status403Forbidden);
```

> [!tip] Practical Tip
> Minimal APIs are excellent for microservices, small APIs, and prototyping. For larger applications with many endpoints, controllers provide better organization through grouping related actions in a class. Neither approach is inherently superior -- choose based on your application's scale and team preferences.

> [!summary] Section Summary
> - Minimal APIs define endpoints as inline delegates with `MapGet()`, `MapPost()`, etc.
> - They support route templates, parameter binding, dependency injection, and return types.
> - Metadata (authorization, names, tags) is added via fluent chaining.
> - Minimal APIs are ideal for small services; controllers scale better for large applications.
