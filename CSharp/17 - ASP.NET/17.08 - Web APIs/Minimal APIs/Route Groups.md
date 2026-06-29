---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


**Route groups** (.NET 7+) let you define a common prefix, filters, and metadata for a set of endpoints. They are created with `app.MapGroup()`.

### Basic Route Group

```csharp
var app = WebApplication.CreateBuilder(args).Build();

// Create a group with shared prefix
var products = app.MapGroup("/api/products");

products.MapGet("/", async (IProductService svc) =>
    Results.Ok(await svc.GetAllAsync()));                  // GET /api/products

products.MapGet("/{id}", async (int id, IProductService svc) =>
    Results.Ok(await svc.GetByIdAsync(id)));               // GET /api/products/{id}

products.MapPost("/", async (CreateProductDto dto, IProductService svc) =>
{
    var product = await svc.CreateAsync(dto);
    return Results.Created($"/api/products/{product.Id}", product);
});                                                         // POST /api/products

products.MapPut("/{id}", async (int id, UpdateProductDto dto, IProductService svc) =>
    Results.Ok(await svc.UpdateAsync(id, dto)));           // PUT /api/products/{id}

products.MapDelete("/{id}", async (int id, IProductService svc) =>
{
    await svc.DeleteAsync(id);
    return Results.NoContent();
});                                                         // DELETE /api/products/{id}

app.Run();
```

### Nested Groups

```csharp
var api = app.MapGroup("/api");

var products = api.MapGroup("/products");
products.MapGet("/", GetAllProducts);
products.MapGet("/{id}", GetProductById);

var orders = api.MapGroup("/orders");
orders.MapGet("/", GetAllOrders);
orders.MapGet("/{id}", GetOrderById);

// Results in: /api/products, /api/products/{id}, /api/orders, /api/orders/{id}
```

### Shared Metadata and Filters on Groups

```csharp
var admin = app.MapGroup("/api/admin")
    .RequireAuthorization("AdminPolicy")   // All endpoints require admin auth
    .AddEndpointFilter<ApiKeyFilter>()     // All endpoints go through API key filter
    .WithTags("Admin");                    // OpenAPI tag for all endpoints

admin.MapGet("/users", GetAllUsers);
admin.MapDelete("/users/{id}", DeleteUser);
admin.MapGet("/audit-log", GetAuditLog);
```

### Organizing Groups with Extension Methods

For clean code, define groups as extension methods:

```csharp
// In ProductEndpoints.cs
public static class ProductEndpoints
{
    public static RouteGroupBuilder MapProductEndpoints(this WebApplication app)
    {
        var group = app.MapGroup("/api/products")
            .WithTags("Products");

        group.MapGet("/", GetAll);
        group.MapGet("/{id}", GetById);
        group.MapPost("/", Create);
        group.MapPut("/{id}", Update);
        group.MapDelete("/{id}", Delete);

        return group;
    }

    private static async Task<IResult> GetAll(IProductService svc) =>
        Results.Ok(await svc.GetAllAsync());

    private static async Task<IResult> GetById(int id, IProductService svc) =>
        await svc.GetByIdAsync(id) is Product p
            ? Results.Ok(p)
            : Results.NotFound();

    private static async Task<IResult> Create(
        CreateProductDto dto, IProductService svc)
    {
        var product = await svc.CreateAsync(dto);
        return Results.Created($"/api/products/{product.Id}", product);
    }

    private static async Task<IResult> Update(
        int id, UpdateProductDto dto, IProductService svc) =>
        Results.Ok(await svc.UpdateAsync(id, dto));

    private static async Task<IResult> Delete(int id, IProductService svc)
    {
        await svc.DeleteAsync(id);
        return Results.NoContent();
    }
}

// In Program.cs -- one line per feature area
app.MapProductEndpoints();
app.MapOrderEndpoints();
app.MapCategoryEndpoints();
```

> [!tip]
> This extension method pattern is the recommended way to organize minimal APIs at scale. Each feature area gets its own file with a `Map{Feature}Endpoints()` method, keeping `Program.cs` focused on configuration.

> [!summary] Section Summary
> Route groups define a shared prefix, metadata, and filters for related endpoints. They support nesting and are best organized using extension methods on `WebApplication`. Each feature area should have its own `Map{Feature}Endpoints()` method for clean separation.
