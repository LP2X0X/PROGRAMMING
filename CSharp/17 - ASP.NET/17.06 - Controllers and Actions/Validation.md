---
tags:
 - csharp
 - asp-net-core
 - validation
 - model-binding
---

# Validation

```ad-note
ASP.NET Core provides a layered validation system that catches invalid data before it reaches your business logic. At its simplest, you decorate model properties with **Data Annotation** attributes and let the framework validate automatically. For more complex scenarios, you can implement **IValidatableObject** for cross-property rules, create **custom validation attributes**, or use third-party libraries like **FluentValidation**. This note covers all of these approaches, from basic annotations through advanced validation patterns.
```

---

## Table of Contents

- [Server-Side Validation with Data Annotations](#Server-Side%20Validation%20with%20Data%20Annotations)
- [Common Data Annotation Attributes](#Common%20Data%20Annotation%20Attributes)
- [ModelState.IsValid](#ModelState.IsValid)
- [ApiController Automatic Validation](#ApiController%20Automatic%20Validation)
- [Custom Validation Attributes](#Custom%20Validation%20Attributes)
- [IValidatableObject -- Class-Level Validation](#IValidatableObject%20--%20Class-Level%20Validation)
- [Validation Error Messages](#Validation%20Error%20Messages)
- [Client-Side Validation (MVC Views)](#Client-Side%20Validation%20(MVC%20Views))
- [FluentValidation (Third-Party)](#FluentValidation%20(Third-Party))
- [Manual Validation](#Manual%20Validation)
- [Validation in Minimal APIs](#Validation%20in%20Minimal%20APIs)
- [Real-World Example](#Real-World%20Example)
- [Comprehensive Summary](#Comprehensive%20Summary)
- [Related Topics](#Related%20Topics)

---

## Server-Side Validation with Data Annotations

**Data Annotations** are attributes from the `System.ComponentModel.DataAnnotations` namespace. You apply them to model properties to declare validation rules. When a request comes in:

1. The **model binder** populates the model from the request data (see [[Model Binding]])
2. The binder runs **validation** against every Data Annotation attribute on the model
3. Validation results are collected into **`ModelState`**, a dictionary of property-level errors
4. Your controller action (or the framework) inspects `ModelState` to decide how to respond

```csharp
using System.ComponentModel.DataAnnotations;

public class CreateProductRequest
{
    [Required]
    [StringLength(200, MinimumLength = 3)]
    public string Name { get; set; } = string.Empty;

    [Range(0.01, 99999.99)]
    public decimal Price { get; set; }

    [Required]
    [EmailAddress]
    public string ContactEmail { get; set; } = string.Empty;
}
```

The framework handles validation automatically -- you do not call any validation method yourself. By the time your action method executes, `ModelState` already contains any validation errors.

```ad-warning
Value types like `int`, `decimal`, and `bool` always have a value (their default: `0`, `0m`, `false`). Applying `[Required]` to a non-nullable value type does **not** ensure the client actually sent a value -- it only checks that the value is not `null`. To truly require a client-supplied value, use **nullable types**: `int?`, `decimal?`, or `bool?` with `[Required]`.
```

```ad-summary
Data Annotations are attributes from `System.ComponentModel.DataAnnotations` applied to model properties. The model binder runs validation automatically after binding and stores errors in `ModelState`. Value types need nullable wrappers for `[Required]` to work meaningfully.
```

---

## Common Data Annotation Attributes

The following table lists the most commonly used validation attributes. All live in `System.ComponentModel.DataAnnotations` unless noted otherwise.

| Attribute | Purpose | Example Usage |
|---|---|---|
| `[Required]` | Property must have a non-null value | `[Required] public string Name { get; set; }` |
| `[StringLength(max)]` | Maximum string length (optionally minimum) | `[StringLength(100, MinimumLength = 2)]` |
| `[MinLength(n)]` | Minimum length for strings and collections | `[MinLength(1)] public List<string> Tags { get; set; }` |
| `[MaxLength(n)]` | Maximum length for strings and collections | `[MaxLength(50)] public string Code { get; set; }` |
| `[Range(min, max)]` | Numeric range; also works with `DateTime` | `[Range(1, 1000)] public int Quantity { get; set; }` |
| `[EmailAddress]` | Validates email format | `[EmailAddress] public string Email { get; set; }` |
| `[Phone]` | Validates phone number format | `[Phone] public string PhoneNumber { get; set; }` |
| `[Url]` | Validates URL format | `[Url] public string Website { get; set; }` |
| `[RegularExpression(pattern)]` | Custom regex pattern match | `[RegularExpression(@"^[A-Z]{2}\d{4}$")]` |
| `[Compare("OtherProperty")]` | Must match another property's value | `[Compare("Password")] public string ConfirmPassword { get; set; }` |
| `[CreditCard]` | Credit card format (Luhn algorithm) | `[CreditCard] public string CardNumber { get; set; }` |
| `[DataType(DataType.Password)]` | UI rendering hint (NOT a validator) | `[DataType(DataType.Password)] public string Password { get; set; }` |
| `[Display(Name = "...")]` | Friendly name in error messages | `[Display(Name = "Product Name")]` |
| `[AllowedValues]` (.NET 8+) | Restrict to specific values | `[AllowedValues("S", "M", "L", "XL")]` |
| `[DeniedValues]` (.NET 8+) | Exclude specific values | `[DeniedValues("admin", "root", "system")]` |
| `[Length(min, max)]` (.NET 8+) | Combined min/max for strings and collections | `[Length(1, 10)] public List<int> Ids { get; set; }` |

```ad-attention
`[DataType]` is **not** a validation attribute. `[DataType(DataType.Password)]` tells UI frameworks (like Tag Helpers) to render a password input, but it performs **zero** validation. If you want format validation, use `[RegularExpression]`, `[EmailAddress]`, `[Phone]`, `[Url]`, or `[CreditCard]` instead.
```

### Range with Dates

`[Range]` works with dates by specifying string boundaries and the type:

```csharp
[Range(typeof(DateTime), "2020-01-01", "2030-12-31",
    ErrorMessage = "Date must be between 2020 and 2030")]
public DateTime OrderDate { get; set; }
```

### AllowedValues and DeniedValues (.NET 8+)

These attributes were introduced in .NET 8 to restrict property values to a specific set:

```csharp
public class ShippingRequest
{
    [Required]
    [AllowedValues("Standard", "Express", "Overnight",
        ErrorMessage = "Shipping method must be Standard, Express, or Overnight")]
    public string ShippingMethod { get; set; } = string.Empty;

    [Required]
    [DeniedValues("TestUser", "Admin",
        ErrorMessage = "Username cannot be a reserved name")]
    public string Username { get; set; } = string.Empty;

    // Length replaces separate MinLength + MaxLength
    [Length(1, 5)]
    public List<string> RecipientNames { get; set; } = new();
}
```

```ad-summary
Data Annotations cover most common validation needs: required fields, string lengths, numeric ranges, format validation (email, phone, URL, credit card), regex patterns, and property comparison. .NET 8 added `[AllowedValues]`, `[DeniedValues]`, and `[Length]`. Remember that `[DataType]` is a UI hint, not a validator.
```

---

## ModelState.IsValid

When the model binder finishes binding and validation, it stores all errors in `ModelState` -- a `ModelStateDictionary` on the controller base class.

### Basic Pattern

```csharp
[HttpPost]
public IActionResult CreateProduct(CreateProductRequest request)
{
    if (!ModelState.IsValid)
    {
        // Return the form with validation error messages displayed
        return View(request);
    }

    // Business logic -- only reached if all validation passed
    _productService.Create(request);
    return RedirectToAction(nameof(Index));
}
```

For API controllers returning JSON:

```csharp
[HttpPost]
public IActionResult CreateProduct(CreateProductRequest request)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    var product = _productService.Create(request);
    return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
}
```

### Adding Custom Errors

You can add errors to `ModelState` manually for business-rule violations that go beyond attribute-level validation:

```csharp
[HttpPost]
public IActionResult CreateOrder(CreateOrderRequest request)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    // Business rule: check if the product actually exists
    if (!_productService.Exists(request.ProductId))
    {
        ModelState.AddModelError(nameof(request.ProductId),
            "The specified product does not exist.");
        return BadRequest(ModelState);
    }

    // Model-level error (not tied to a specific property)
    if (_orderService.HasReachedDailyLimit(request.CustomerId))
    {
        ModelState.AddModelError(string.Empty,
            "Daily order limit has been reached for this customer.");
        return BadRequest(ModelState);
    }

    var order = _orderService.Create(request);
    return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
}
```

The first parameter of `AddModelError` is the **key** -- use the property name for property-level errors, or `string.Empty` for model-level errors.

### What the Error Response Looks Like

When you return `BadRequest(ModelState)`, the response body is a JSON dictionary of errors keyed by property name:

```json
{
  "Name": ["The Name field is required."],
  "Price": ["The field Price must be between 0.01 and 99999.99."],
  "ContactEmail": ["The ContactEmail field is not a valid e-mail address."]
}
```

```ad-summary
`ModelState` is a `ModelStateDictionary` collecting all binding and validation errors. Check `ModelState.IsValid` to guard your action logic. Use `AddModelError(key, message)` for custom business-rule errors. Pass `string.Empty` as the key for model-level errors not tied to a specific property.
```

---

## ApiController Automatic Validation

When you decorate a controller with `[ApiController]`, the framework adds several API-specific behaviors, one of which is **automatic model validation**. You do **not** need to check `ModelState.IsValid` yourself.

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(CreateProductRequest request)
    {
        // No ModelState.IsValid check needed!
        // If validation fails, the framework returns 400 before this code runs.
        var product = _productService.Create(request);
        return CreatedAtAction(nameof(Get), new { id = product.Id }, product);
    }
}
```

### How It Works

The `[ApiController]` attribute registers a built-in **action filter** called `ModelStateInvalidFilter`. This filter runs before your action method and:

1. Checks if `ModelState.IsValid` is `false`
2. If invalid, short-circuits the pipeline and returns a **400 Bad Request** response
3. The response body is a `ValidationProblemDetails` object (RFC 7807 Problem Details format)

### The Automatic 400 Response

The automatic response follows the Problem Details standard:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": ["The Name field is required."],
    "Price": ["The field Price must be between 0.01 and 99999.99."],
    "ContactEmail": ["The ContactEmail field is not a valid e-mail address."]
  },
  "traceId": "00-abc123def456-789ghi-00"
}
```

```ad-info
The `ValidationProblemDetails` response includes a `traceId` that correlates with your server-side logging, making it easier to debug validation issues in production.
```

### Disabling Automatic Validation

If you need to handle invalid model state yourself (for example, to add business-rule errors before returning), disable the automatic filter:

```csharp
builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.SuppressModelStateInvalidFilter = true;
    });
```

You can also customize the response factory instead of disabling it entirely:

```csharp
builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.InvalidModelStateResponseFactory = context =>
        {
            var problemDetails = new ValidationProblemDetails(context.ModelState)
            {
                Title = "Validation failed",
                Status = StatusCodes.Status422UnprocessableEntity,
                Instance = context.HttpContext.Request.Path
            };

            return new UnprocessableEntityObjectResult(problemDetails);
        };
    });
```

```ad-summary
`[ApiController]` adds automatic model validation via `ModelStateInvalidFilter`. Invalid requests get a 400 response with `ValidationProblemDetails` (RFC 7807) before your action code runs. Disable with `SuppressModelStateInvalidFilter = true`, or customize the response with `InvalidModelStateResponseFactory`.
```

---

## Custom Validation Attributes

When built-in attributes are not enough, create your own by inheriting from `ValidationAttribute`.

### Example: FutureDateAttribute

```csharp
using System.ComponentModel.DataAnnotations;

public class FutureDateAttribute : ValidationAttribute
{
    public FutureDateAttribute()
        : base("The {0} field must be a date in the future.")
    {
    }

    protected override ValidationResult? IsValid(
        object? value, ValidationContext validationContext)
    {
        if (value is null)
        {
            // Let [Required] handle null checks
            return ValidationResult.Success;
        }

        if (value is DateTime dateValue)
        {
            if (dateValue > DateTime.UtcNow)
            {
                return ValidationResult.Success;
            }

            string displayName = validationContext.DisplayName;
            string errorMessage = FormatErrorMessage(displayName);
            return new ValidationResult(errorMessage,
                new[] { validationContext.MemberName! });
        }

        return new ValidationResult("Invalid date value.");
    }
}
```

Usage:

```csharp
public class CreateEventRequest
{
    [Required]
    [StringLength(200)]
    public string EventName { get; set; } = string.Empty;

    [Required]
    [FutureDate]
    public DateTime? EventDate { get; set; }
}
```

```ad-tip
Always use the `IsValid(object? value, ValidationContext validationContext)` overload, not the simpler `IsValid(object? value)` one. The `ValidationContext` gives you access to the property name, the parent object (for cross-property checks), and the DI container.
```

### Example: NotEqualToAttribute

This attribute validates that one property does not equal another -- useful for ensuring "new password" differs from "old password":

```csharp
using System.ComponentModel.DataAnnotations;

public class NotEqualToAttribute : ValidationAttribute
{
    private readonly string _otherPropertyName;

    public NotEqualToAttribute(string otherPropertyName)
        : base("The {0} field must not equal the {1} field.")
    {
        _otherPropertyName = otherPropertyName;
    }

    protected override ValidationResult? IsValid(
        object? value, ValidationContext validationContext)
    {
        if (value is null)
        {
            return ValidationResult.Success;
        }

        // Access the other property via the parent object
        var otherProperty = validationContext.ObjectType
            .GetProperty(_otherPropertyName);

        if (otherProperty is null)
        {
            return new ValidationResult(
                $"Unknown property: {_otherPropertyName}");
        }

        var otherValue = otherProperty.GetValue(validationContext.ObjectInstance);

        if (value.Equals(otherValue))
        {
            string errorMessage = string.Format(
                ErrorMessageString,
                validationContext.DisplayName,
                _otherPropertyName);
            return new ValidationResult(errorMessage,
                new[] { validationContext.MemberName! });
        }

        return ValidationResult.Success;
    }
}
```

Usage:

```csharp
public class ChangePasswordRequest
{
    [Required]
    [DataType(DataType.Password)]
    public string CurrentPassword { get; set; } = string.Empty;

    [Required]
    [StringLength(100, MinimumLength = 8)]
    [DataType(DataType.Password)]
    [NotEqualTo(nameof(CurrentPassword),
        ErrorMessage = "New password must be different from current password.")]
    public string NewPassword { get; set; } = string.Empty;

    [Required]
    [Compare(nameof(NewPassword))]
    [DataType(DataType.Password)]
    public string ConfirmNewPassword { get; set; } = string.Empty;
}
```

### Accessing DI Services in Custom Attributes

The `ValidationContext` provides access to the DI container, so your attribute can resolve services:

```csharp
public class UniqueEmailAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(
        object? value, ValidationContext validationContext)
    {
        if (value is not string email)
        {
            return ValidationResult.Success;
        }

        // Resolve a service from DI
        var userService = validationContext
            .GetRequiredService<IUserService>();

        if (userService.EmailExists(email))
        {
            return new ValidationResult(
                "This email address is already registered.");
        }

        return ValidationResult.Success;
    }
}
```

```ad-warning
Accessing DI services from validation attributes creates a dependency on the DI container being available. This works in ASP.NET Core request pipelines but will fail in unit tests unless you manually configure a `ValidationContext` with a service provider. Consider using [[Filters]] or FluentValidation for complex rules that need DI.
```

```ad-summary
Custom validation attributes inherit from `ValidationAttribute` and override `IsValid(object?, ValidationContext)`. The `ValidationContext` provides access to the property name, the parent object instance (for cross-property comparisons), and the DI container. Let `[Required]` handle null checks -- return `Success` for null values in your custom attribute.
```

---

## IValidatableObject -- Class-Level Validation

When validation rules span multiple properties, implement `IValidatableObject` on the model class itself. This is ideal for **cross-property** rules like "end date must be after start date" or "if payment method is credit card, card number is required."

```csharp
using System.ComponentModel.DataAnnotations;

public class CreateEventRequest : IValidatableObject
{
    [Required]
    [StringLength(200)]
    public string EventName { get; set; } = string.Empty;

    [Required]
    public DateTime? StartDate { get; set; }

    [Required]
    public DateTime? EndDate { get; set; }

    [Range(1, 10000)]
    public int MaxAttendees { get; set; }

    [Range(0, 10000)]
    public int? MinAttendees { get; set; }

    public IEnumerable<ValidationResult> Validate(
        ValidationContext validationContext)
    {
        // This method only runs AFTER all individual property validations pass.

        if (EndDate.HasValue && StartDate.HasValue
            && EndDate.Value <= StartDate.Value)
        {
            yield return new ValidationResult(
                "End date must be after start date.",
                new[] { nameof(EndDate) });
        }

        if (MinAttendees.HasValue && MinAttendees.Value > MaxAttendees)
        {
            yield return new ValidationResult(
                "Minimum attendees cannot exceed maximum attendees.",
                new[] { nameof(MinAttendees), nameof(MaxAttendees) });
        }

        if (StartDate.HasValue && StartDate.Value < DateTime.UtcNow.Date)
        {
            yield return new ValidationResult(
                "Start date cannot be in the past.",
                new[] { nameof(StartDate) });
        }
    }
}
```

```ad-note
The `Validate` method **only runs after** all individual property-level Data Annotations pass. If `[Required]` on `StartDate` fails, `Validate` is never called. This means you can safely assume that individually validated properties have valid values inside `Validate`.
```

### Accessing DI Services

Just like custom attributes, `IValidatableObject.Validate` receives a `ValidationContext` with DI access:

```csharp
public IEnumerable<ValidationResult> Validate(
    ValidationContext validationContext)
{
    var eventService = validationContext
        .GetRequiredService<IEventService>();

    if (eventService.HasConflict(StartDate!.Value, EndDate!.Value))
    {
        yield return new ValidationResult(
            "This time slot conflicts with an existing event.");
    }
}
```

```ad-summary
`IValidatableObject` provides class-level validation via the `Validate` method. It runs after all property-level Data Annotations pass, making it safe to assume property values are individually valid. Use `yield return` to report multiple errors. The `ValidationContext` parameter gives access to DI services.
```

---

## Validation Error Messages

### Default Messages

Every built-in validation attribute has a default error message. The `{0}` placeholder is replaced with the property's display name (from `[Display(Name = "...")]` or the property name itself):

```csharp
// Default message: "The Product Name field is required."
[Required]
[Display(Name = "Product Name")]
public string Name { get; set; } = string.Empty;

// Default message: "The field Quantity must be between 1 and 1000."
[Range(1, 1000)]
public int Quantity { get; set; }
```

### Custom Messages

Override the default with `ErrorMessage`:

```csharp
[Required(ErrorMessage = "Please provide a product name.")]
public string Name { get; set; } = string.Empty;

[Range(1, 1000, ErrorMessage = "Order between 1 and 1000 units.")]
public int Quantity { get; set; }
```

### Parameterized Messages

Validation attributes support `string.Format`-style placeholders. The exact placeholders vary by attribute:

```csharp
// {0} = display name, {1} = max length, {2} = min length
[StringLength(100, MinimumLength = 3,
    ErrorMessage = "{0} must be between {2} and {1} characters.")]
public string Name { get; set; } = string.Empty;

// {0} = display name, {1} = min value, {2} = max value
[Range(0.01, 99999.99,
    ErrorMessage = "{0} must be between {1:C} and {2:C}.")]
public decimal Price { get; set; }
```

### Localization

For multi-language applications, use resource files instead of inline strings:

```csharp
[Required(
    ErrorMessageResourceType = typeof(ValidationMessages),
    ErrorMessageResourceName = nameof(ValidationMessages.NameRequired))]
[Display(
    ResourceType = typeof(DisplayNames),
    Name = nameof(DisplayNames.ProductName))]
public string Name { get; set; } = string.Empty;
```

This reads the error message and display name from `.resx` resource files, enabling translation without changing code.

```ad-summary
Default error messages use `{0}` for the property display name. Override with `ErrorMessage` for custom text or use format placeholders like `{1}` and `{2}` for attribute-specific values (max length, min value, etc.). For localization, point to `.resx` resource files via `ErrorMessageResourceType` and `ErrorMessageResourceName`.
```

---

## Client-Side Validation (MVC Views)

ASP.NET Core MVC supports **client-side validation** via the **jQuery Unobtrusive Validation** library. This prevents form submission before the request even reaches the server, providing immediate feedback to users.

### How It Works

1. Tag Helpers generate `data-val-*` HTML attributes from your Data Annotations
2. The jQuery Unobtrusive Validation library reads these attributes at runtime
3. When the user submits the form, client-side validation runs first
4. If validation fails, the form does not submit and error messages appear next to the fields

### Required Scripts

Include these scripts (typically via a partial or layout):

```html
<script src="~/lib/jquery/dist/jquery.min.js"></script>
<script src="~/lib/jquery-validation/dist/jquery.validate.min.js"></script>
<script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"></script>
```

### Razor Form with Tag Helpers

```csharp
@model CreateProductRequest

<form asp-action="Create" asp-controller="Products" method="post">
    <!-- Validation summary for model-level errors -->
    <div asp-validation-summary="ModelOnly" class="text-danger"></div>

    <div class="form-group">
        <label asp-for="Name"></label>
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="Price"></label>
        <input asp-for="Price" class="form-control" />
        <span asp-validation-for="Price" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="ContactEmail"></label>
        <input asp-for="ContactEmail" class="form-control" />
        <span asp-validation-for="ContactEmail" class="text-danger"></span>
    </div>

    <button type="submit" class="btn btn-primary">Create</button>
</form>
```

The generated HTML includes validation metadata:

```html
<input class="form-control" type="text" id="Name" name="Name"
    data-val="true"
    data-val-required="The Name field is required."
    data-val-length="The Name must be between 3 and 200 characters."
    data-val-length-max="200"
    data-val-length-min="3" />
<span class="text-danger field-validation-valid"
    data-valmsg-for="Name"
    data-valmsg-replace="true"></span>
```

### Validation Summary Modes

The `asp-validation-summary` tag helper has three modes:

| Mode | Behavior |
|---|---|
| `None` | No summary displayed |
| `ModelOnly` | Shows only model-level errors (key = `string.Empty`) |
| `All` | Shows all errors -- model-level and property-level |

```ad-danger
Client-side validation is a **UX convenience**, not a security measure. It can be bypassed by disabling JavaScript, using browser dev tools, or calling your API directly with cURL/Postman. **Always validate on the server.** Client-side validation reduces unnecessary round trips but must never be your only line of defense.
```

```ad-summary
jQuery Unobtrusive Validation reads `data-val-*` attributes generated from Data Annotations to provide instant client-side feedback. Use `asp-validation-for` for per-field errors and `asp-validation-summary` for model-level errors. Client-side validation is a UX feature -- always validate server-side as well.
```

---

## FluentValidation (Third-Party)

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

---

## Manual Validation

Sometimes you need to validate a model that was not bound from an HTTP request -- for example, a model constructed in code or loaded from a database.

### TryValidateModel (Inside Controllers)

```csharp
[HttpPost]
public IActionResult ImportProducts(IFormFile file)
{
    var products = _csvParser.Parse<CreateProductRequest>(file);

    foreach (var product in products)
    {
        // Clear previous errors and validate this model
        ModelState.Clear();

        if (!TryValidateModel(product))
        {
            return BadRequest(new
            {
                Error = $"Validation failed for product: {product.Name}",
                Details = ModelState
            });
        }
    }

    _productService.BulkCreate(products);
    return Ok();
}
```

### Validator.TryValidateObject (Outside Controllers)

For validation outside the controller pipeline (services, background jobs, console apps):

```csharp
using System.ComponentModel.DataAnnotations;

public static class ValidationHelper
{
    public static (bool IsValid, List<ValidationResult> Errors) Validate<T>(T model)
        where T : notnull
    {
        var context = new ValidationContext(model);
        var results = new List<ValidationResult>();

        // The 'true' parameter validates ALL properties, not just [Required]
        bool isValid = Validator.TryValidateObject(
            model, context, results, validateAllProperties: true);

        return (isValid, results);
    }
}

// Usage in a background service
public class OrderProcessingService
{
    public void ProcessOrder(CreateOrderRequest request)
    {
        var (isValid, errors) = ValidationHelper.Validate(request);

        if (!isValid)
        {
            throw new ValidationException(
                string.Join("; ", errors.Select(e => e.ErrorMessage)));
        }

        // Process the valid order...
    }
}
```

```ad-attention
The `validateAllProperties` parameter (the last `bool` argument) defaults to `false` if omitted. When `false`, only `[Required]` attributes are checked. **Always pass `true`** to validate all Data Annotation attributes on the object.
```

```ad-summary
Use `TryValidateModel` inside controllers when you construct models in code. Use `Validator.TryValidateObject` outside the controller pipeline (services, background jobs). Always pass `validateAllProperties: true` to check all attributes, not just `[Required]`.
```

---

## Validation in Minimal APIs

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

---

## Real-World Example

Putting it all together: a `CreateOrderRequest` model with comprehensive validation using Data Annotations, `IValidatableObject`, and the corresponding controller action.

### The Model

```csharp
using System.ComponentModel.DataAnnotations;

public class CreateOrderRequest : IValidatableObject
{
    [Required(ErrorMessage = "Customer name is required.")]
    [StringLength(200, MinimumLength = 2,
        ErrorMessage = "Customer name must be between 2 and 200 characters.")]
    [Display(Name = "Customer Name")]
    public string CustomerName { get; set; } = string.Empty;

    [Required(ErrorMessage = "Email address is required.")]
    [EmailAddress(ErrorMessage = "Please provide a valid email address.")]
    public string Email { get; set; } = string.Empty;

    [Required]
    [Phone]
    [Display(Name = "Phone Number")]
    public string PhoneNumber { get; set; } = string.Empty;

    [Required(ErrorMessage = "Product ID is required.")]
    public Guid? ProductId { get; set; }

    [Required]
    [Range(1, 10000, ErrorMessage = "Quantity must be between 1 and 10,000.")]
    public int? Quantity { get; set; }

    [Required]
    [StringLength(500)]
    [Display(Name = "Shipping Address")]
    public string ShippingAddress { get; set; } = string.Empty;

    [StringLength(1000)]
    [Display(Name = "Special Instructions")]
    public string? SpecialInstructions { get; set; }

    public bool IsGiftOrder { get; set; }

    [StringLength(200)]
    [Display(Name = "Gift Message")]
    public string? GiftMessage { get; set; }

    [Required]
    [AllowedValues("Standard", "Express", "Overnight",
        ErrorMessage = "Shipping method must be Standard, Express, or Overnight.")]
    [Display(Name = "Shipping Method")]
    public string ShippingMethod { get; set; } = string.Empty;

    [Required]
    public DateTime? RequestedDeliveryDate { get; set; }

    // Cross-property validation
    public IEnumerable<ValidationResult> Validate(
        ValidationContext validationContext)
    {
        // Gift message required for gift orders
        if (IsGiftOrder && string.IsNullOrWhiteSpace(GiftMessage))
        {
            yield return new ValidationResult(
                "A gift message is required for gift orders.",
                new[] { nameof(GiftMessage) });
        }

        // Delivery date must be in the future
        if (RequestedDeliveryDate.HasValue
            && RequestedDeliveryDate.Value.Date <= DateTime.UtcNow.Date)
        {
            yield return new ValidationResult(
                "Requested delivery date must be in the future.",
                new[] { nameof(RequestedDeliveryDate) });
        }

        // Express/Overnight requires at least 1 day lead time,
        // Standard requires at least 3 days
        if (RequestedDeliveryDate.HasValue)
        {
            int minimumLeadDays = ShippingMethod switch
            {
                "Overnight" => 1,
                "Express" => 2,
                "Standard" => 3,
                _ => 3
            };

            var earliestDelivery = DateTime.UtcNow.Date.AddDays(minimumLeadDays);

            if (RequestedDeliveryDate.Value.Date < earliestDelivery)
            {
                yield return new ValidationResult(
                    $"{ShippingMethod} shipping requires at least " +
                    $"{minimumLeadDays} business day(s) lead time.",
                    new[] { nameof(RequestedDeliveryDate) });
            }
        }
    }
}
```

### The Controller Action

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly IProductService _productService;

    public OrdersController(
        IOrderService orderService,
        IProductService productService)
    {
        _orderService = orderService;
        _productService = productService;
    }

    [HttpPost]
    [ProducesResponseType(typeof(OrderResponse), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ValidationProblemDetails),
        StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> Create(CreateOrderRequest request)
    {
        // Data Annotations + IValidatableObject already validated by [ApiController].
        // Add business-rule validation that requires service calls.

        var product = await _productService.GetByIdAsync(request.ProductId!.Value);

        if (product is null)
        {
            ModelState.AddModelError(nameof(request.ProductId),
                "The specified product does not exist.");
            return ValidationProblem();
        }

        if (product.Stock < request.Quantity!.Value)
        {
            ModelState.AddModelError(nameof(request.Quantity),
                $"Insufficient stock. Only {product.Stock} units available.");
            return ValidationProblem();
        }

        var order = await _orderService.CreateAsync(request);

        return CreatedAtAction(
            nameof(GetById),
            new { id = order.Id },
            OrderResponse.FromEntity(order));
    }

    [HttpGet("{id:guid}")]
    public async Task<IActionResult> GetById(Guid id)
    {
        var order = await _orderService.GetByIdAsync(id);
        return order is null
            ? NotFound()
            : Ok(OrderResponse.FromEntity(order));
    }
}
```

### Example Validation Error Responses

**Missing required fields:**

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "CustomerName": ["Customer name is required."],
    "Email": ["Email address is required."],
    "ProductId": ["The ProductId field is required."],
    "Quantity": ["The Quantity field is required."],
    "ShippingAddress": ["The ShippingAddress field is required."],
    "ShippingMethod": ["Shipping method must be Standard, Express, or Overnight."],
    "RequestedDeliveryDate": ["The RequestedDeliveryDate field is required."]
  },
  "traceId": "00-a1b2c3d4e5f6-789abc-00"
}
```

**Cross-property and business-rule errors:**

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "GiftMessage": ["A gift message is required for gift orders."],
    "RequestedDeliveryDate": [
      "Standard shipping requires at least 3 business day(s) lead time."
    ],
    "Quantity": ["Insufficient stock. Only 5 units available."]
  },
  "traceId": "00-f6e5d4c3b2a1-def012-00"
}
```

```ad-summary
A real-world model combines `[Required]`, `[StringLength]`, `[Range]`, `[EmailAddress]`, `[AllowedValues]`, and other Data Annotations for property-level rules. `IValidatableObject` handles cross-property rules (gift message requirement, delivery lead time). The `[ApiController]` attribute handles the annotation-level validation automatically, while business-rule errors (stock check, product existence) are added to `ModelState` in the action method.
```

---

## Comprehensive Summary

| Approach | Best For | Runs Automatically | Testable |
|---|---|---|---|
| **Data Annotations** | Simple property-level rules | Yes (MVC + `[ApiController]`) | Limited |
| **IValidatableObject** | Cross-property rules on a model | Yes (after annotations pass) | Medium |
| **Custom Attributes** | Reusable single-property rules | Yes | Medium |
| **FluentValidation** | Complex business rules, async | Yes (with registration) | Easy |
| **Manual (`Validator.TryValidateObject`)** | Non-HTTP scenarios | No (you call it) | Easy |
| **Endpoint Filters** | Minimal API validation | Yes (once registered) | Easy |

Key principles:

- **Always validate server-side.** Client-side validation is a UX convenience, not a security boundary.
- **Use nullable types** with `[Required]` for value types to detect truly missing values.
- **`[ApiController]`** removes the need for manual `ModelState.IsValid` checks in web API controllers.
- **`IValidatableObject.Validate`** only runs after all property-level annotations pass.
- **Minimal APIs** have no automatic validation -- use endpoint filters.
- **FluentValidation** shines when rules are complex, need DI, or must be unit tested.
- **Data Annotations and FluentValidation** can coexist in the same project.

---

## Related Topics

- [[Controllers Overview]] -- Controller fundamentals and routing
- [[Action Results]] -- Returning responses from actions
- [[Model Binding]] -- How request data becomes C# objects (the step before validation)
- [[Filters]] -- Action filters, including custom validation filters
- [[17.08 - Web APIs]] -- Web API patterns that rely on validation