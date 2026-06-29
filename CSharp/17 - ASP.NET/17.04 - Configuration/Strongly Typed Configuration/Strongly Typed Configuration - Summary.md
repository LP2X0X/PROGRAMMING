---
tags:
  - csharp
  - asp-net-core
  - configuration
  - binding
  - strongly-typed
---


> [!tip] Complete Summary
> **Strongly typed configuration** in ASP.NET Core replaces magic strings with compile-time-safe C# classes that are populated automatically from JSON, environment variables, and other providers.
>
> **Key takeaways:**
>
> 1. **Binding** maps configuration sections to C# classes via `Get<T>()`, `Bind()`, or `BindConfiguration()`
> 2. **`services.Configure<T>()`** integrates with DI and unlocks the full [[Options Pattern]] (IOptions, IOptionsSnapshot, IOptionsMonitor)
> 3. **Nested objects** work by recursive property-tree walking; always initialize nested properties
> 4. **Arrays/Lists** bind positionally; dictionaries bind by key
> 5. **Enums** bind by name (case-insensitive) and fail silently on mismatch
> 6. **Data annotations** (`[Required]`, `[Range]`, `[Url]`) enforce constraints when paired with `ValidateDataAnnotations()`
> 7. **`ValidateOnStart()`** is essential for fail-fast behavior; without it, validation only runs on first access
> 8. **Custom validation** uses `Validate()` lambdas or `IValidateOptions<T>` (the latter supports DI)
> 9. **Change tokens** via `GetReloadToken()` enable runtime config watching; `IOptionsMonitor<T>` handles this automatically
> 10. **Records** with `init` properties provide immutable, strongly typed options; avoid positional parameter syntax
>
> **The production pattern:**
> ```csharp
> builder.Services.AddOptions<MySettings>()
>     .BindConfiguration("SectionName")
>     .ValidateDataAnnotations()
>     .ValidateOnStart();
> ```
