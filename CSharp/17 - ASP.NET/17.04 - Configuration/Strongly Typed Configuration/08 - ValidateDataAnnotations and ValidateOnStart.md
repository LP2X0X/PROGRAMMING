---
tags:
  - csharp
  - asp-net-core
  - configuration
  - binding
  - strongly-typed
---


Annotations on their own do **nothing** unless you wire up validation during service registration.

### Basic Validation Registration

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations();
```

> [!danger] Critical Warning
> Without `ValidateDataAnnotations()`, your `[Required]` and `[Range]` attributes are completely ignored at runtime. The application will happily start with invalid configuration. Always pair annotations with explicit validation registration.

### Fail-Fast with ValidateOnStart

By default, options validation only runs **the first time `IOptions<T>.Value` is accessed**. This means a misconfigured app might start successfully and only crash later when the faulty options are first resolved.

**`ValidateOnStart()`** forces validation to run during application startup, so misconfigurations are caught immediately:

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

If validation fails at startup, you get an `OptionsValidationException` with a clear message:

```
Unhandled exception. Microsoft.Extensions.Options.OptionsValidationException:
DataAnnotation validation failed for 'SmtpSettings' members: 'Host' with the
error: 'SMTP Host is required'.
```

> [!tip] Pro Tip
> **Always use `ValidateOnStart()`** in production applications. Failing fast at startup is vastly preferable to discovering bad configuration at 3 AM when a rarely-used code path first tries to read the setting.

### Full Pattern

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")          // Binds to the "Smtp" section
    .ValidateDataAnnotations()           // Enables annotation-based validation
    .ValidateOnStart();                  // Runs validation at startup

builder.Services.AddOptions<DatabaseSettings>()
    .BindConfiguration("Database")
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

> [!ad-note]
> `BindConfiguration("SectionName")` is a shorthand that combines `GetSection()` and `Bind()`. It is equivalent to calling `Configure<T>(config.GetSection("SectionName"))` but works within the `AddOptions<T>()` fluent API.

> [!summary] Section Summary
> `ValidateDataAnnotations()` activates annotation-based validation. `ValidateOnStart()` triggers validation at startup instead of on first access. Always use both together for fail-fast behavior.
