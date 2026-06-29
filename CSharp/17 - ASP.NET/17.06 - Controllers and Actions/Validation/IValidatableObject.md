---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


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
