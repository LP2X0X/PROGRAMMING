---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


The `@functions` directive (or `@code` in Blazor) lets you define methods and properties inside the view:

```cshtml
@functions {
    public string GetCssClass(decimal price)
    {
        if (price > 100) return "text-danger fw-bold";
        if (price > 50) return "text-warning";
        return "text-success";
    }

    public string FormatDiscount(decimal original, decimal discounted)
    {
        var percentage = (1 - discounted / original) * 100;
        return $"{percentage:F0}% off";
    }
}

@foreach (var product in Model.Products)
{
    <div class="product">
        <span class="@GetCssClass(product.Price)">@product.Price.ToString("C")</span>
        @if (product.DiscountedPrice.HasValue)
        {
            <span class="discount">
                @FormatDiscount(product.Price, product.DiscountedPrice.Value)
            </span>
        }
    </div>
}
```

> [!warning] Common Misconception
> Just because you *can* define methods in views does not mean you *should*. `@functions` is appropriate for small formatting helpers specific to one view. If the logic is reusable or complex, it belongs in a view model, extension method, or [[Partial Views and View Components|view component]]. Overuse of `@functions` leads to "fat views" that are hard to test and maintain.

> [!summary] Section Summary
> - `@functions { }` defines C# methods and properties inside a view
> - Appropriate for small, view-specific formatting helpers
> - Complex or reusable logic should live in view models or services
> - `@code { }` is the Blazor equivalent (not used in MVC views)

---
