---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


Minimal APIs do ==not include built-in model validation== like the `[ApiController]` attribute provides for controllers. You must handle validation explicitly.

### Manual Validation

```csharp
app.MapPost("/products", async (CreateProductDto dto, IProductService svc) =>
{
    var errors = new Dictionary<string, string[]>();

    if (string.IsNullOrWhiteSpace(dto.Name))
        errors["Name"] = new[] { "Name is required." };
    else if (dto.Name.Length > 200)
        errors["Name"] = new[] { "Name cannot exceed 200 characters." };

    if (dto.Price <= 0)
        errors["Price"] = new[] { "Price must be greater than zero." };

    if (string.IsNullOrWhiteSpace(dto.Sku))
        errors["Sku"] = new[] { "SKU is required." };

    if (errors.Count > 0)
        return Results.ValidationProblem(errors);

    var product = await svc.CreateAsync(dto);
    return Results.Created($"/api/products/{product.Id}", product);
});
```

### Validation with FluentValidation

A more scalable approach uses **FluentValidation** with an endpoint filter:

```csharp
// Install: dotnet add package FluentValidation.DependencyInjectionExtensions
```

```csharp
// Validator definition
public class CreateProductValidator : AbstractValidator<CreateProductDto>
{
    public CreateProductValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Product name is required.")
            .MaximumLength(200).WithMessage("Name cannot exceed 200 characters.");

        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Price must be greater than zero.");

        RuleFor(x => x.Sku)
            .NotEmpty().WithMessage("SKU is required.")
            .Matches(@"^[A-Z]{2}-\d{4}$")
            .WithMessage("SKU must be in format XX-0000.");

        RuleFor(x => x.CategoryId)
            .GreaterThan(0).WithMessage("Valid category is required.");
    }
}
```

```csharp
// Generic validation filter using FluentValidation
public class FluentValidationFilter<T> : IEndpointFilter where T : class
{
    public async ValueTask<object?> InvokeAsync(
        EndpointFilterInvocationContext context,
        EndpointFilterDelegate next)
    {
        var dto = context.Arguments.OfType<T>().FirstOrDefault();

        if (dto is null)
            return Results.BadRequest("Request body is required.");

        var validator = context.HttpContext.RequestServices
            .GetService<IValidator<T>>();

        if (validator is not null)
        {
            var result = await validator.ValidateAsync(dto);
            if (!result.IsValid)
            {
                var errors = result.Errors
                    .GroupBy(e => e.PropertyName)
                    .ToDictionary(
                        g => g.Key,
                        g => g.Select(e => e.ErrorMessage).ToArray());

                return Results.ValidationProblem(errors);
            }
        }

        return await next(context);
    }
}
```

```csharp
// Registration
builder.Services.AddValidatorsFromAssemblyContaining<CreateProductValidator>();

// Usage
app.MapPost("/products", CreateProduct)
    .AddEndpointFilter<FluentValidationFilter<CreateProductDto>>();

app.MapPut("/products/{id}", UpdateProduct)
    .AddEndpointFilter<FluentValidationFilter<UpdateProductDto>>();
```

### Validation with Data Annotations and MiniValidator

```csharp
// Install: dotnet add package MiniValidation
using MiniValidation;

public record CreateProductDto(
    [Required, StringLength(200)] string Name,
    [Range(0.01, 999999.99)] decimal Price,
    [Required, RegularExpression(@"^[A-Z]{2}-\d{4}$")] string Sku);

app.MapPost("/products", async (CreateProductDto dto, IProductService svc) =>
{
    if (!MiniValidator.TryValidate(dto, out var errors))
        return Results.ValidationProblem(errors);

    var product = await svc.CreateAsync(dto);
    return Results.Created($"/api/products/{product.Id}", product);
});
```

> [!warning]
> Unlike controllers with `[ApiController]`, minimal APIs do not automatically validate Data Annotation attributes. You must invoke validation explicitly, either manually, via a library like MiniValidation/FluentValidation, or through an endpoint filter.

> [!summary] Section Summary
> Minimal APIs require explicit validation since there is no automatic model validation. Options include manual validation, FluentValidation with endpoint filters (recommended for complex APIs), or MiniValidation for Data Annotations. Always return `Results.ValidationProblem()` for structured error responses.
