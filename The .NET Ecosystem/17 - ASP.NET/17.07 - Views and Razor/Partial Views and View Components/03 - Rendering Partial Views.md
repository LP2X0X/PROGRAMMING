---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


There are several ways to render a partial view. The **partial tag helper** is the modern, recommended approach.

### Partial Tag Helper (Recommended)

```cshtml
<partial name="_ProductCard" model="product" />
```

### Html.PartialAsync (Older Approach)

```cshtml
@await Html.PartialAsync("_ProductCard", product)
```

### Html.RenderPartialAsync (Writes Directly to Response)

```cshtml
@{ await Html.RenderPartialAsync("_ProductCard", product); }
```

> [!tip] Practical Tip
> Prefer the `<partial>` tag helper for clarity and consistency with the rest of the [[Tag Helpers]] ecosystem. `Html.PartialAsync` returns an `IHtmlContent` (buffered in memory), while `Html.RenderPartialAsync` writes directly to the output stream. For most cases the performance difference is negligible, but `RenderPartialAsync` can be slightly more efficient for very large partials.

### Rendering in a Loop

```cshtml
<div class="row">
    @foreach (var product in Model.Products)
    {
        <div class="col-md-4 mb-4">
            <partial name="_ProductCard" model="product" />
        </div>
    }
</div>
```

> [!warning] Common Misconception
> `Html.Partial()` (synchronous, without `Async`) exists but can cause deadlocks in ASP.NET Core. Always use the async variants or the `<partial>` tag helper. The synchronous methods are obsolete and will generate compiler warnings.

> [!summary] Section Summary
> - `<partial name="..." model="..." />` is the recommended way to render partials
> - `Html.PartialAsync()` and `Html.RenderPartialAsync()` are the programmatic alternatives
> - Always use async methods -- synchronous `Html.Partial()` risks deadlocks
> - Partials work naturally inside loops for rendering collections

---
