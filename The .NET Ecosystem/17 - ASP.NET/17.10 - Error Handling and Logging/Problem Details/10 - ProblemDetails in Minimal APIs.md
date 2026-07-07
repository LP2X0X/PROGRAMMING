---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - problem-details
  - api
---


Minimal APIs have their own helpers for generating ProblemDetails responses:

```csharp
// Results.Problem() -- returns ProblemDetails
app.MapGet("/api/products/{id}", (int id) =>
{
    var product = repository.GetById(id);

    return product is not null
        ? Results.Ok(product)
        : Results.Problem(
            type: "https://example.com/errors/product-not-found",
            title: "Product Not Found",
            statusCode: StatusCodes.Status404NotFound,
            detail: $"No product exists with ID {id}.",
            instance: $"/api/products/{id}");
});

// TypedResults.Problem() -- for typed return values
app.MapGet("/api/products/{id}", Results<Ok<Product>, ProblemHttpResult> (int id) =>
{
    var product = repository.GetById(id);

    return product is not null
        ? TypedResults.Ok(product)
        : TypedResults.Problem(
            title: "Product Not Found",
            statusCode: StatusCodes.Status404NotFound,
            detail: $"No product exists with ID {id}.");
});

// Results.ValidationProblem() -- for validation errors
app.MapPost("/api/products", (CreateProductRequest request) =>
{
    var errors = Validate(request);

    if (errors.Count > 0)
    {
        return Results.ValidationProblem(errors,
            title: "Validation Failed",
            detail: "One or more fields have invalid values.");
    }

    var product = repository.Create(request);
    return Results.Created($"/api/products/{product.Id}", product);
});
```

## ProblemDetails with Filters in Minimal APIs

```csharp
// Global endpoint filter that catches domain exceptions
app.MapGroup("/api")
    .AddEndpointFilter(async (context, next) =>
    {
        try
        {
            return await next(context);
        }
        catch (NotFoundException ex)
        {
            return Results.Problem(
                title: "Not Found",
                statusCode: 404,
                detail: ex.Message);
        }
        catch (ConflictException ex)
        {
            return Results.Problem(
                title: "Conflict",
                statusCode: 409,
                detail: ex.Message);
        }
    });
```

> [!summary] Section Summary
> - `Results.Problem()` and `TypedResults.Problem()` generate ProblemDetails in minimal APIs
> - `Results.ValidationProblem()` generates `ValidationProblemDetails` with the `errors` dictionary
> - Use `TypedResults` for OpenAPI documentation support (the return types are visible to the schema generator)
> - Endpoint filters can catch domain exceptions and convert them to ProblemDetails for a group of endpoints
