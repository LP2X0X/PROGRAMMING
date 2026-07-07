---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


### Strongly-Typed Model (Recommended)

The partial declares a `@model` directive and receives a typed object:

```cshtml
@* _ProductCard.cshtml *@
@model Product

<h3>@Model.Name</h3>
<p>@Model.Price.ToString("C")</p>
```

```cshtml
@* Calling view *@
<partial name="_ProductCard" model="myProduct" />
```

### ViewData / ViewBag

Partial views inherit the parent view's `ViewData` dictionary. You can also pass additional `ViewData`:

```cshtml
<partial name="_ProductCard"
         model="myProduct"
         view-data="@(new ViewDataDictionary(ViewData) { { "ShowAddToCart", true } })" />
```

```cshtml
@* Inside _ProductCard.cshtml *@
@if ((bool?)ViewData["ShowAddToCart"] == true)
{
    <button class="btn btn-primary">Add to Cart</button>
}
```

> [!warning] Common Misconception
> Partial views do NOT inherit the parent's `@model` automatically. If the partial has `@model Product` and you do not pass a model, `Model` will be `null`. The partial's `Model` property is independent of the parent view's `Model`.

> [!tip] Practical Tip
> Prefer strongly-typed models over `ViewData` for partials. Create a dedicated view model if a partial needs both a domain object and display flags:
> ```csharp
> public class ProductCardViewModel
> {
>     public Product Product { get; set; }
>     public bool ShowAddToCart { get; set; }
>     public bool ShowPrice { get; set; }
> }
> ```

> [!summary] Section Summary
> - Pass data to partials using the `model` attribute on the `<partial>` tag helper
> - Partials do NOT automatically inherit the parent's `@model` -- always pass data explicitly
> - `ViewData` can be passed for additional flags, but prefer dedicated view models
> - Strongly-typed models provide IntelliSense and compile-time safety

---
