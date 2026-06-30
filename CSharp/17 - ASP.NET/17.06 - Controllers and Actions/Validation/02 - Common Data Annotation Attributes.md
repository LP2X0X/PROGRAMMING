---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


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
