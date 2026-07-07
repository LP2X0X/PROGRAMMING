---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


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
