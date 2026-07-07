---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


**FluentValidation** is a popular library that replaces (or supplements) Data Annotations with separate validator classes. It provides a fluent API, better testability, and more powerful rule composition.

Install the NuGet packages:

```
dotnet add package FluentValidation
dotnet add package FluentValidation.AspNetCore
```

### Basic Validator Class

```csharp
using FluentValidation;

public class CreateOrderRequestValidator
    : AbstractValidator<CreateOrderRequest>
{
    public CreateOrderRequestValidator()
    {
        RuleFor(x => x.CustomerName)
            .NotEmpty()
                .WithMessage("Customer name is required.")
            .MaximumLength(200)
                .WithMessage("Customer name cannot exceed 200 characters.");

        RuleFor(x => x.Email)
            .NotEmpty()
            .EmailAddress()
                .WithMessage("A valid email address is required.");

        RuleFor(x => x.Quantity)
            .GreaterThan(0)
                .WithMessage("Quantity must be at least 1.")
            .LessThanOrEqualTo(10000)
                .WithMessage("Quantity cannot exceed 10,000.");

        RuleFor(x => x.ShippingAddress)
            .NotEmpty()
            .MaximumLength(500);

        RuleFor(x => x.ProductId)
            .NotEmpty()
                .WithMessage("Please select a product.");
    }
}
```

### Conditional Rules

```csharp
public class CreateOrderRequestValidator
    : AbstractValidator<CreateOrderRequest>
{
    public CreateOrderRequestValidator()
    {
        RuleFor(x => x.CustomerName).NotEmpty().MaximumLength(200);

        // Only validate Country when IsInternational is true
        When(x => x.IsInternational, () =>
        {
            RuleFor(x => x.Country)
                .NotEmpty()
                    .WithMessage("Country is required for international orders.");

            RuleFor(x => x.CustomsDeclaration)
                .NotEmpty()
                    .WithMessage("Customs declaration is required for international orders.");
        });

        // The Otherwise block runs when the condition is false
        Otherwise(() =>
        {
            RuleFor(x => x.State)
                .NotEmpty()
                    .WithMessage("State is required for domestic orders.");
        });
    }
}
```

### Custom Rules with Must

```csharp
RuleFor(x => x.OrderDate)
    .Must(date => date >= DateTime.UtcNow.Date)
        .WithMessage("Order date cannot be in the past.");

RuleFor(x => x.EndDate)
    .GreaterThan(x => x.StartDate)
        .WithMessage("End date must be after start date.");

// Must with full model access
RuleFor(x => x.DiscountCode)
    .Must((order, discountCode) =>
    {
        // Complex validation logic with access to the entire model
        return order.Quantity >= 10 || string.IsNullOrEmpty(discountCode);
    })
    .WithMessage("Discount codes require a minimum quantity of 10.");
```

### Common Built-In Validators

| Validator | Purpose |
|---|---|
| `NotEmpty()` | Not null, not empty string, not default value |
| `NotNull()` | Not null (allows empty strings) |
| `MaximumLength(n)` | String max length |
| `MinimumLength(n)` | String min length |
| `Length(min, max)` | String length range |
| `EmailAddress()` | Valid email format |
| `GreaterThan(n)` | Numeric greater than |
| `LessThan(n)` | Numeric less than |
| `GreaterThanOrEqualTo(n)` | Numeric >= |
| `LessThanOrEqualTo(n)` | Numeric <= |
| `InclusiveBetween(min, max)` | Numeric range (inclusive) |
| `Matches(regex)` | Regex match |
| `Must(predicate)` | Custom predicate |
| `When(condition, rules)` | Conditional rules |
| `SetValidator(validator)` | Nested object validation |

### Registration and Integration

Register FluentValidation in `Program.cs`:

```csharp
using FluentValidation;
using FluentValidation.AspNetCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Register all validators from the assembly
builder.Services.AddFluentValidationAutoValidation();
builder.Services.AddValidatorsFromAssemblyContaining<CreateOrderRequestValidator>();
```

With this setup, FluentValidation automatically integrates with the ASP.NET Core validation pipeline. When a request comes in, the corresponding validator runs and errors appear in `ModelState` just like Data Annotation errors.

### DI in Validators

Because validators are registered with DI, you can inject services:

```csharp
public class CreateOrderRequestValidator
    : AbstractValidator<CreateOrderRequest>
{
    public CreateOrderRequestValidator(IProductService productService)
    {
        RuleFor(x => x.ProductId)
            .NotEmpty()
            .MustAsync(async (productId, cancellationToken) =>
            {
                return await productService.ExistsAsync(productId, cancellationToken);
            })
            .WithMessage("The specified product does not exist.");
    }
}
```

### Why FluentValidation over Data Annotations

| Aspect | Data Annotations | FluentValidation |
|---|---|---|
| **Location** | On the model class itself | Separate validator class |
| **Testability** | Hard to unit test | Easy -- just instantiate the validator |
| **Separation of concerns** | Model knows its validation rules | Validation logic is decoupled |
| **Complex rules** | Custom attributes needed | Fluent API with `When`, `Must`, `Unless` |
| **Async validation** | Not supported | `MustAsync`, `WhenAsync` |
| **DI support** | Limited (via ValidationContext) | Constructor injection |
| **Conditional logic** | Awkward | First-class with `When` / `Unless` |
| **Reusability** | Limited to attributes | Compose validators, include child validators |

```ad-tip
You can use both Data Annotations and FluentValidation in the same project. Data Annotations work well for simple rules (`[Required]`, `[StringLength]`), while FluentValidation handles complex business rules. They both feed into `ModelState`.
```

```ad-summary
FluentValidation provides separate validator classes with a fluent API for defining rules. It integrates with ASP.NET Core via `AddFluentValidationAutoValidation()` and feeds errors into `ModelState`. Key advantages over Data Annotations: testability, separation of concerns, conditional rules (`When`/`Unless`), async validation (`MustAsync`), and constructor-injected DI.
```
