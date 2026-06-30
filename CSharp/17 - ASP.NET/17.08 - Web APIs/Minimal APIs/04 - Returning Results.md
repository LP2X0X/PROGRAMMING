---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


Minimal API handlers can return various types. The framework serializes them appropriately based on the return type.

### Implicit Returns

```csharp
// String -> text/plain 200
app.MapGet("/hello", () => "Hello, World!");

// Object -> application/json 200
app.MapGet("/product", () => new Product { Id = 1, Name = "Widget" });

// IEnumerable -> application/json 200
app.MapGet("/products", () => new List<Product>
{
    new(1, "Widget"),
    new(2, "Gadget")
});
```

### The `Results` Static Class

The `Results` class provides factory methods for common HTTP responses:

```csharp
app.MapGet("/products/{id}", async (int id, IProductService service) =>
{
    var product = await service.GetByIdAsync(id);
    return product is not null
        ? Results.Ok(product)           // 200 with JSON body
        : Results.NotFound();           // 404
});

app.MapPost("/products", async (CreateProductDto dto, IProductService service) =>
{
    var product = await service.CreateAsync(dto);
    return Results.Created(             // 201 with Location header
        $"/products/{product.Id}",
        product);
});

app.MapDelete("/products/{id}", async (int id, IProductService service) =>
{
    var deleted = await service.DeleteAsync(id);
    return deleted
        ? Results.NoContent()           // 204
        : Results.NotFound();           // 404
});
```

### Common `Results` Methods

| Method                       | Status Code | Use Case                          |
|---|---|---|
| `Results.Ok(value)`          | 200         | Successful response with body     |
| `Results.Created(uri, val)`  | 201         | Resource created                  |
| `Results.Accepted(uri)`      | 202         | Accepted for processing           |
| `Results.NoContent()`        | 204         | Success with no body              |
| `Results.BadRequest(val)`    | 400         | Invalid request                   |
| `Results.Unauthorized()`     | 401         | Not authenticated                 |
| `Results.Forbid()`           | 403         | Not authorized                    |
| `Results.NotFound(val)`      | 404         | Resource not found                |
| `Results.Conflict(val)`      | 409         | Conflict with current state       |
| `Results.UnprocessableEntity(val)` | 422   | Validation failure                |
| `Results.Problem(detail)`    | 500         | Server error (ProblemDetails)     |
| `Results.ValidationProblem(errors)` | 400  | Validation errors (ProblemDetails)|
| `Results.File(bytes, ct)`    | 200         | File download                     |
| `Results.Stream(stream)`     | 200         | Stream response                   |
| `Results.Redirect(url)`      | 302         | Redirect                          |
| `Results.Json(val)`          | 200         | JSON with custom options          |
| `Results.Text(content)`      | 200         | Plain text                        |

### Returning ProblemDetails

ASP.NET Core uses **ProblemDetails** (RFC 7807) for standardized error responses. See [[Content Negotiation]] for how response formats are determined.

```csharp
app.MapGet("/products/{id}", async (int id, IProductService service) =>
{
    if (id <= 0)
    {
        return Results.Problem(
            detail: "Product ID must be a positive integer.",
            statusCode: 400,
            title: "Invalid Product ID");
    }

    var product = await service.GetByIdAsync(id);
    return product is not null
        ? Results.Ok(product)
        : Results.Problem(
            detail: $"No product found with ID {id}.",
            statusCode: 404,
            title: "Product Not Found");
});
```

### Returning Validation Errors

```csharp
app.MapPost("/products", async (CreateProductDto dto, IProductService service) =>
{
    var errors = new Dictionary<string, string[]>();

    if (string.IsNullOrWhiteSpace(dto.Name))
        errors["Name"] = new[] { "Product name is required." };

    if (dto.Price <= 0)
        errors["Price"] = new[] { "Price must be greater than zero." };

    if (errors.Count > 0)
        return Results.ValidationProblem(errors);

    var product = await service.CreateAsync(dto);
    return Results.Created($"/products/{product.Id}", product);
});
```

> [!summary] Section Summary
> Handlers can return strings, objects, or `IResult` values. The `Results` class provides factory methods for all standard HTTP responses including `Ok`, `NotFound`, `Created`, `BadRequest`, `ValidationProblem`, and `Problem`. Use `Results.ValidationProblem` for structured validation errors following RFC 7807.
