---
tags:
  - csharp
  - asp-net-core
  - configuration
  - binding
  - strongly-typed
---


For validation rules that go beyond what data annotations can express, use the **`Validate()` method** in the fluent options API.

### Simple Custom Validation

```csharp
builder.Services.AddOptions<RetrySettings>()
    .BindConfiguration("AppSettings:Retry")
    .ValidateDataAnnotations()
    .Validate(options => options.MaxAttempts > 0,
        "MaxRetries must be positive")
    .Validate(options => options.DelayMilliseconds >= 100,
        "Delay must be at least 100ms")
    .ValidateOnStart();
```

### Multi-Property Validation

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()
    .Validate(options =>
    {
        if (options.UseSsl && options.Port == 25)
            return false; // Port 25 should not be used with SSL
        return true;
    }, "Port 25 cannot be used with SSL enabled. Use port 465 or 587.")
    .ValidateOnStart();
```

### IValidateOptions\<T\> for Complex Scenarios

For validation logic that requires injected services (e.g., checking a database or external system), implement `IValidateOptions<T>`:

```csharp
public class SmtpSettingsValidator : IValidateOptions<SmtpSettings>
{
    public ValidateOptionsResult Validate(string? name, SmtpSettings options)
    {
        var errors = new List<string>();

        if (string.IsNullOrWhiteSpace(options.Host))
            errors.Add("Host is required.");

        if (options.Port is < 1 or > 65535)
            errors.Add("Port must be between 1 and 65535.");

        if (options.UseSsl && options.Port == 25)
            errors.Add("Port 25 is not compatible with SSL.");

        if (!options.Host.Contains('.'))
            errors.Add("Host must be a fully qualified domain name.");

        return errors.Count > 0
            ? ValidateOptionsResult.Fail(errors)
            : ValidateOptionsResult.Success;
    }
}
```

Register the validator:

```csharp
builder.Services.AddSingleton<IValidateOptions<SmtpSettings>,
    SmtpSettingsValidator>();

builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateOnStart();
```

> [!tip] Pro Tip
> `IValidateOptions<T>` validators are resolved from DI, so they can depend on other services. This is the only validation approach that supports dependency injection. The `Validate()` lambda cannot inject services.

### Combining All Validation Approaches

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()                     // Annotations first
    .Validate(o => o.Port != 25 || !o.UseSsl,      // Inline lambda
        "Port 25 + SSL is invalid")
    .ValidateOnStart();                            // Fail fast

// Plus an IValidateOptions<T> registered separately
builder.Services.AddSingleton<
    IValidateOptions<SmtpSettings>,
    SmtpSettingsValidator>();
```

All three validation mechanisms run. If any fails, the aggregated errors are reported.

> [!summary] Section Summary
> Use `Validate()` lambdas for simple cross-property checks. Implement `IValidateOptions<T>` when validation needs injected services. All validation approaches (annotations, lambdas, IValidateOptions) combine and run together.
