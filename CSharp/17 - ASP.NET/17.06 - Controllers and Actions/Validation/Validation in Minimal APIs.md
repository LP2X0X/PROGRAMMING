---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


**Minimal APIs do not have automatic model validation.** Unlike `[ApiController]`-decorated controllers, minimal API endpoints do not run Data Annotation validation or check `ModelState`. You must add validation yourself.

### Manual Validation in the Endpoint

The simplest approach -- validate directly inside the handler:

```csharp
app.MapPost("/api/products", (CreateProductRequest request) =>
{
    var context = new ValidationContext(request);
    var errors = new List<ValidationResult>();

    if (!Validator.TryValidateObject(request, context, errors, true))
    {
        return Results.ValidationProblem(
            errors.GroupBy(e => e.MemberNames.FirstOrDefault() ?? string.Empty)
                  .ToDictionary(
                      g => g.Key,
                      g => g.Select(e => e.ErrorMessage!).ToArray()));
    }

    // Process valid request...
    return Results.Created($"/api/products/{product.Id}", product);
});
```

### Endpoint Filter Approach (Reusable)

A cleaner solution is an **endpoint filter** that validates any request type with Data Annotations:

```csharp
public class ValidationFilter<T> : IEndpointFilter where T : class
{
    public async ValueTask<object?> InvokeAsync(
        EndpointFilterInvocationContext context,
        EndpointFilterDelegate next)
    {
        // Find the argument of type T
        var model = context.Arguments
            .OfType<T>()
            .FirstOrDefault();

        if (model is null)
        {
            return Results.BadRequest("Request body is required.");
        }

        var validationContext = new ValidationContext(model);
        var errors = new List<ValidationResult>();

        if (!Validator.TryValidateObject(
            model, validationContext, errors, validateAllProperties: true))
        {
            var problemDetails = errors
                .GroupBy(e => e.MemberNames.FirstOrDefault() ?? string.Empty)
                .ToDictionary(
                    g => g.Key,
                    g => g.Select(e => e.ErrorMessage!).ToArray());

            return Results.ValidationProblem(problemDetails);
        }

        return await next(context);
    }
}
```

Register it on endpoints:

```csharp
app.MapPost("/api/products", (CreateProductRequest request) =>
{
    // Validation already passed -- safe to proceed
    var product = productService.Create(request);
    return Results.Created($"/api/products/{product.Id}", product);
})
.AddEndpointFilter<ValidationFilter<CreateProductRequest>>();

app.MapPost("/api/orders", (CreateOrderRequest request) =>
{
    var order = orderService.Create(request);
    return Results.Created($"/api/orders/{order.Id}", order);
})
.AddEndpointFilter<ValidationFilter<CreateOrderRequest>>();
```

### FluentValidation with Minimal APIs

FluentValidation also works with endpoint filters:

```csharp
public class FluentValidationFilter<T> : IEndpointFilter where T : class
{
    private readonly IValidator<T> _validator;

    public FluentValidationFilter(IValidator<T> validator)
    {
        _validator = validator;
    }

    public async ValueTask<object?> InvokeAsync(
        EndpointFilterInvocationContext context,
        EndpointFilterDelegate next)
    {
        var model = context.Arguments.OfType<T>().FirstOrDefault();

        if (model is null)
        {
            return Results.BadRequest("Request body is required.");
        }

        var result = await _validator.ValidateAsync(model);

        if (!result.IsValid)
        {
            var errors = result.Errors
                .GroupBy(e => e.PropertyName)
                .ToDictionary(
                    g => g.Key,
                    g => g.Select(e => e.ErrorMessage).ToArray());

            return Results.ValidationProblem(errors);
        }

        return await next(context);
    }
}
```

### AsParameters for Complex Binding

`[AsParameters]` lets minimal APIs bind complex types from query strings and route parameters, but it still does not run validation automatically:

```csharp
app.MapGet("/api/products", ([AsParameters] ProductSearchQuery query) =>
{
    // query is populated from query string parameters
    // but NOT validated -- you still need a filter
    return productService.Search(query);
})
.AddEndpointFilter<ValidationFilter<ProductSearchQuery>>();

public class ProductSearchQuery
{
    [Range(1, int.MaxValue)]
    public int Page { get; set; } = 1;

    [Range(1, 100)]
    public int PageSize { get; set; } = 20;

    [StringLength(200)]
    public string? SearchTerm { get; set; }
}
```

```ad-summary
Minimal APIs have no built-in validation. Use `Validator.TryValidateObject` for manual validation, or create reusable endpoint filters (`IEndpointFilter`) that validate models before the handler runs. `[AsParameters]` binds complex types from query/route data but does not trigger validation. FluentValidation integrates via custom endpoint filters with DI-injected validators.
```
