---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


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
