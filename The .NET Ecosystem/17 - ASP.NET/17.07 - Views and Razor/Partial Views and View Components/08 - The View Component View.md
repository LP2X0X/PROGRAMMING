---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


View component views follow a specific file location convention:

```
/Views/Shared/Components/[ComponentName]/Default.cshtml
```

Or, for controller-specific view components:

```
/Views/[Controller]/Components/[ComponentName]/Default.cshtml
```

For the `CartSummaryViewComponent`:

```cshtml
@* /Views/Shared/Components/CartSummary/Default.cshtml *@
@model CartSummaryViewModel

<div class="cart-summary">
    <a asp-controller="Cart" asp-action="Index" class="nav-link">
        <span class="cart-icon">Cart</span>
        @if (Model.ItemCount > 0)
        {
            <span class="badge bg-danger">@Model.ItemCount</span>
            <span class="cart-total">@Model.TotalPrice.ToString("C")</span>
        }
        else
        {
            <span class="text-muted">Empty</span>
        }
    </a>
</div>
```

**Naming the view:**
- `Default.cshtml` is the default view name
- You can return a named view: `return View("Mini", model);` looks for `Mini.cshtml` in the same folder
- This allows multiple display modes for the same view component

```csharp
public async Task<IViewComponentResult> InvokeAsync(string displayMode = "default")
{
    var model = await BuildModel();
    return displayMode switch
    {
        "mini" => View("Mini", model),
        "full" => View("Full", model),
        _ => View(model)  // Default.cshtml
    };
}
```

> [!summary] Section Summary
> - View component views live in `/Views/Shared/Components/[Name]/Default.cshtml`
> - `Default.cshtml` is the default name; named views allow multiple display modes
> - The view receives a strongly-typed model from `InvokeAsync()` via `View(model)`
> - View component views support all [[Razor Syntax]] features, tag helpers, and nested partials

---
