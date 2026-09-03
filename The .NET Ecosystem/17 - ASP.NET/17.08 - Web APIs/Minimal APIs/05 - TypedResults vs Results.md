---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


.NET 7 introduced `TypedResults`, which returns concrete types instead of `IResult`. This seemingly small difference has a major impact on OpenAPI documentation and testability.

### The Problem with `Results`

```csharp
// Returns IResult - OpenAPI doesn't know the response schema
app.MapGet("/products/{id}", async (int id, IProductService service) =>
{
    var product = await service.GetByIdAsync(id);
    return product is not null
        ? Results.Ok(product)       // Returns IResult
        : Results.NotFound();       // Returns IResult
});
```

With `Results`, the return type is `IResult` -- the framework cannot infer what types or status codes the endpoint produces, so OpenAPI documentation is incomplete.

### The Solution: `TypedResults`

```csharp
// Returns concrete types - OpenAPI knows exactly what to expect
app.MapGet("/products/{id}", async Task<Results<Ok<Product>, NotFound>> (
    int id, IProductService service) =>
{
    var product = await service.GetByIdAsync(id);
    return product is not null
        ? TypedResults.Ok(product)       // Returns Ok<Product>
        : TypedResults.NotFound();       // Returns NotFound
});
```

> [!warning]
> The `Results<T1, T2, ...>` union return type in the method signature is what enables OpenAPI inference. Without it, even `TypedResults` methods will not fully describe the endpoint. The union type can hold up to 6 type parameters: `Results<T1, T2, T3, T4, T5, T6>`.

### Side-by-Side Comparison

```csharp
// Using Results (IResult) -- no OpenAPI metadata inferred
app.MapGet("/v1/products/{id}", async (int id, IProductService svc) =>
{
    var product = await svc.GetByIdAsync(id);
    return product is not null
        ? Results.Ok(product)
        : Results.NotFound();
});

// Using TypedResults -- full OpenAPI metadata inferred
app.MapGet("/v2/products/{id}", async Task<Results<Ok<Product>, NotFound>> (
    int id, IProductService svc) =>
{
    var product = await svc.GetByIdAsync(id);
    return product is not null
        ? TypedResults.Ok(product)
        : TypedResults.NotFound();
});
```

### Testing Benefits

`TypedResults` makes unit testing cleaner because you get concrete types:

```csharp
[Fact]
public async Task GetProduct_ReturnsOk_WhenProductExists()
{
    // Arrange
    var service = new FakeProductService();
    service.Add(new Product(1, "Widget", 9.99m));

    // Act -- call the handler directly
    var result = await ProductEndpoints.GetById(1, service);

    // Assert -- result is a concrete type, not IResult
    var okResult = Assert.IsType<Ok<Product>>(result.Result);
    Assert.Equal("Widget", okResult.Value!.Name);
}
```

### When to Use Which

| Scenario                                  | Use              |
|---|---|
| Prototype / quick endpoint                | `Results`        |
| API with OpenAPI/Swagger documentation    | `TypedResults`   |
| Endpoints you want to unit test cleanly   | `TypedResults`   |
| Internal/private microservice API         | Either           |
| Public-facing API                         | `TypedResults`   |

> [!summary] Section Summary
> `TypedResults` returns concrete types (`Ok<T>`, `NotFound`, `Created<T>`) instead of `IResult`, enabling automatic OpenAPI metadata inference and cleaner unit tests. Use `Results<T1, T2, ...>` as the return type to express all possible responses. Prefer `TypedResults` for any API that needs documentation or testability.
