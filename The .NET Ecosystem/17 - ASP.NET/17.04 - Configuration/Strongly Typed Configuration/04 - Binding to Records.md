---
tags:
  - csharp
  - asp-net-core
  - configuration
  - binding
  - strongly-typed
---


Starting with .NET 6, configuration binding works with **C# records** and **init-only properties**.

### Record-Based Options

```csharp
public record SmtpSettings
{
    public required string Host { get; init; }
    public int Port { get; init; } = 587;
    public required string Username { get; init; }
    public required string Password { get; init; }
    public bool UseSsl { get; init; } = true;
}
```

### Registration (same as with classes)

```csharp
builder.Services.AddOptions<SmtpSettings>()
    .BindConfiguration("Smtp")
    .ValidateDataAnnotations()
    .ValidateOnStart();
```

### Why Use Records for Configuration

- **Immutability**: `init` setters prevent accidental modification after binding
- **Value equality**: Two `SmtpSettings` with the same values are considered equal
- **Conciseness**: `required` keyword ensures all mandatory properties are set
- **Pattern matching**: Records work naturally with C# pattern matching

> [!ad-note]
> Records with **positional parameters** (constructor-based syntax like `record SmtpSettings(string Host, int Port)`) do **not** work with configuration binding. The binder requires parameterless construction with settable properties. Use the `{ get; init; }` property syntax instead.

> [!example] Record With Validation
> ```csharp
> public record DatabaseOptions
> {
>     [Required]
>     public required string ConnectionString { get; init; }
> 
>     [Range(1, 300)]
>     public int CommandTimeoutSeconds { get; init; } = 30;
> 
>     [Range(1, 200)]
>     public int MaxPoolSize { get; init; } = 100;
> 
>     public bool EnableDetailedErrors { get; init; } = false;
> }
> ```

> [!summary] Section Summary
> Records with `init` properties work with configuration binding and provide immutability. Avoid positional parameter records (constructor syntax) as they are incompatible with the binder.
