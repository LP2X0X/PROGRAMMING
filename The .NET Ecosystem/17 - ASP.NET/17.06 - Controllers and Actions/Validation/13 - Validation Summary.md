---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


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

## Related Topics

- [[Controllers Overview]] -- Controller fundamentals and routing
- [[Action Results]] -- Returning responses from actions
- [[Model Binding]] -- How request data becomes C# objects (the step before validation)
- [[Filters]] -- Action filters, including custom validation filters
- [[17.08 - Web APIs]] -- Web API patterns that rely on validation
