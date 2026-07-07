---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


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
