---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


Every minimal API endpoint is created by calling a `Map{Verb}` method on the `WebApplication` instance. The method takes a **route pattern** and a **handler delegate**.

### HTTP Verb Mappings

```csharp
var app = WebApplication.CreateBuilder(args).Build();

// GET request
app.MapGet("/products", () => "List of products");

// POST request
app.MapPost("/products", () => "Create a product");

// PUT request
app.MapPut("/products/{id}", (int id) => $"Update product {id}");

// DELETE request
app.MapDelete("/products/{id}", (int id) => $"Delete product {id}");

// PATCH request
app.MapPatch("/products/{id}", (int id) => $"Patch product {id}");

// Match any HTTP method
app.MapMethods("/products/report", new[] { "OPTIONS", "HEAD" }, () => "Custom methods");

// Map to all HTTP methods
app.MapFallback(() => Results.NotFound("No matching route"));

app.Run();
```

### Handler Delegate Types

The handler can be a lambda, a local function, a method group, or any `Delegate`:

```csharp
// Lambda expression
app.MapGet("/lambda", () => "From lambda");

// Named local function
app.MapGet("/local", GetMessage);
string GetMessage() => "From local function";

// Static method reference
app.MapGet("/static", ProductHandlers.GetAll);

// Instance method reference
var handler = new ProductHandler();
app.MapGet("/instance", handler.GetAll);
```

```csharp
// Separating handlers into a static class
public static class ProductHandlers
{
    public static IResult GetAll()
    {
        return Results.Ok(new[] { "Product A", "Product B" });
    }

    public static IResult GetById(int id)
    {
        return id > 0
            ? Results.Ok($"Product {id}")
            : Results.BadRequest("Invalid ID");
    }
}
```

> [!tip]
> For anything beyond a few endpoints, organize handlers into static classes grouped by domain. This keeps `Program.cs` clean while retaining the minimal API style.

### Async Handlers

Handlers can be asynchronous. The framework awaits them automatically:

```csharp
app.MapGet("/products", async (IProductRepository repo) =>
{
    var products = await repo.GetAllAsync();
    return Results.Ok(products);
});
```

> [!summary] Section Summary
> Minimal API endpoints are defined using `Map{Verb}` methods on `WebApplication`. Handlers can be lambdas, method groups, or delegates. Async handlers are fully supported. Organize handlers into separate static classes for maintainability.
