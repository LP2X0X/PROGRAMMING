---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


The **input tag helper** is arguably the most powerful built-in tag helper. It uses `asp-for` to generate the `name`, `id`, `type`, and `data-val-*` validation attributes from the model property and its data annotations.

```cshtml
@model ProductViewModel

<input asp-for="Name" class="form-control" />
```

Given this model:

```csharp
public class ProductViewModel
{
    [Required(ErrorMessage = "Product name is required")]
    [StringLength(100, MinimumLength = 3)]
    [Display(Name = "Product Name")]
    public string Name { get; set; }
}
```

Renders:

```html
<input class="form-control"
       type="text"
       id="Name"
       name="Name"
       value=""
       data-val="true"
       data-val-required="Product name is required"
       data-val-length="The field Product Name must be a string with a minimum length of 3 and a maximum length of 100."
       data-val-length-max="100"
       data-val-length-min="3" />
```

### Type Inference

The tag helper infers the HTML `type` attribute from the C# property type:

| C# Type / Annotation | HTML type |
|---|---|
| `string` | `text` |
| `int`, `long`, `decimal`, `double` | `number` |
| `bool` | `checkbox` |
| `DateTime` | `datetime-local` |
| `[DataType(DataType.Password)]` | `password` |
| `[DataType(DataType.EmailAddress)]` | `email` |
| `[DataType(DataType.Url)]` | `url` |
| `[DataType(DataType.PhoneNumber)]` | `tel` |
| `[DataType(DataType.Date)]` | `date` |
| `[DataType(DataType.Time)]` | `time` |
| `byte[]` | `file` |
| `[HiddenInput]` | `hidden` |

> [!tip] Practical Tip
> Always apply `[DataType]` annotations on your model for proper input types. A `string Email` property renders as `type="text"` unless you annotate it with `[DataType(DataType.EmailAddress)]` or `[EmailAddress]`, which gives you `type="email"` and mobile-friendly keyboards.

> [!summary] Section Summary
> - `asp-for` binds an input to a model property, generating `name`, `id`, `type`, and validation attributes
> - Data annotations on the model property drive both the HTML type and client-side validation
> - Type inference maps C# types and `[DataType]` annotations to the correct HTML `type`
> - The generated `data-val-*` attributes enable jQuery Unobtrusive Validation

---
