---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


### Label Tag Helper

```cshtml
<label asp-for="Name"></label>
@* Renders: <label for="Name">Product Name</label> *@
@* The text comes from [Display(Name = "...")] or the property name *@
```

### Select Tag Helper

```cshtml
<select asp-for="CategoryId" asp-items="Model.Categories" class="form-select">
    <option value="">-- Select Category --</option>
</select>
```

Where `Model.Categories` is a `SelectList` or `IEnumerable<SelectListItem>`:

```csharp
public class ProductViewModel
{
    public int CategoryId { get; set; }
    public SelectList Categories { get; set; }
}

// In the controller:
var viewModel = new ProductViewModel
{
    Categories = new SelectList(
        await _context.Categories.ToListAsync(),
        "Id",       // value field
        "Name"      // text field
    )
};
```

For enum-based selects:

```cshtml
<select asp-for="Status" asp-items="Html.GetEnumSelectList<OrderStatus>()" class="form-select">
</select>
```

### Textarea Tag Helper

```cshtml
<textarea asp-for="Description" class="form-control" rows="5"></textarea>
@* Generates name, id, and validation attributes just like the input tag helper *@
```

> [!summary] Section Summary
> - `<label asp-for>` generates `for` attribute and display text from model metadata
> - `<select asp-for asp-items>` generates a dropdown from a `SelectList` or `IEnumerable<SelectListItem>`
> - `Html.GetEnumSelectList<T>()` creates select items from an enum type
> - `<textarea asp-for>` generates name, id, and validation attributes from the model property

---
