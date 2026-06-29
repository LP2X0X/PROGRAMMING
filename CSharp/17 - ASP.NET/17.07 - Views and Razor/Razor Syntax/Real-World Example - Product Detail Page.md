---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


Putting it all together -- a complete product detail view using the syntax covered above:

```cshtml
@model MyApp.ViewModels.ProductDetailViewModel
@inject IStringLocalizer<ProductResource> Localizer

@{
    ViewData["Title"] = Model.Product.Name;
    var hasDiscount = Model.Product.DiscountedPrice.HasValue;
    var savingsPercent = hasDiscount
        ? (1 - Model.Product.DiscountedPrice.Value / Model.Product.Price) * 100
        : 0;
}

<nav aria-label="breadcrumb">
    <ol class="breadcrumb">
        <li class="breadcrumb-item">
            <a asp-controller="Home" asp-action="Index">Home</a>
        </li>
        <li class="breadcrumb-item">
            <a asp-controller="Products" asp-action="Index">Products</a>
        </li>
        <li class="breadcrumb-item">
            <a asp-controller="Products" asp-action="ByCategory"
               asp-route-id="@Model.Product.CategoryId">
                @Model.Product.CategoryName
            </a>
        </li>
        <li class="breadcrumb-item active">@Model.Product.Name</li>
    </ol>
</nav>

<div class="row">
    <div class="col-md-6">
        <img src="@Model.Product.ImageUrl"
             alt="@Model.Product.Name"
             class="img-fluid rounded" />
    </div>
    <div class="col-md-6">
        <h1>@Model.Product.Name</h1>
        <p class="text-muted">@Model.Product.Brand</p>

        @if (hasDiscount)
        {
            <p>
                <span class="text-decoration-line-through text-muted">
                    @Model.Product.Price.ToString("C")
                </span>
                <span class="text-danger fs-4 fw-bold ms-2">
                    @Model.Product.DiscountedPrice.Value.ToString("C")
                </span>
                <span class="badge bg-danger ms-2">
                    Save @(savingsPercent.ToString("F0"))%
                </span>
            </p>
        }
        else
        {
            <p class="fs-4 fw-bold">@Model.Product.Price.ToString("C")</p>
        }

        <div class="product-description">
            @Html.Raw(Model.Product.SanitizedDescriptionHtml)
        </div>

        @if (Model.Product.StockCount > 0)
        {
            <p class="text-success">
                @Localizer["InStock"] (@Model.Product.StockCount @Localizer["Available"])
            </p>
            <form asp-controller="Cart" asp-action="Add" method="post">
                <input type="hidden" asp-for="Product.Id" />
                <div class="input-group mb-3" style="max-width: 200px;">
                    <input type="number" name="quantity" value="1"
                           min="1" max="@Model.Product.StockCount"
                           class="form-control" />
                    <button type="submit" class="btn btn-primary">
                        @Localizer["AddToCart"]
                    </button>
                </div>
            </form>
        }
        else
        {
            <p class="text-danger">@Localizer["OutOfStock"]</p>
        }
    </div>
</div>

@* Related products section *@
@if (Model.RelatedProducts.Any())
{
    <hr />
    <h2>@Localizer["RelatedProducts"]</h2>
    <div class="row">
        @foreach (var related in Model.RelatedProducts)
        {
            <div class="col-md-3 mb-3">
                <partial name="_ProductCard" model="related" />
            </div>
        }
    </div>
}

@section Scripts {
    <script>
        var productConfig = @Json.Serialize(new {
            productId = Model.Product.Id,
            maxQuantity = Model.Product.StockCount
        });
    </script>
}
```

This example demonstrates:
- `@model` directive with a strongly-typed view model
- `@inject` for localization
- Code blocks for computed values
- `@if`/`@else` for conditional rendering
- `@foreach` for iteration
- Implicit and explicit expressions
- `@Html.Raw()` for pre-sanitized HTML
- `@Json.Serialize()` for passing data to JavaScript
- [[Tag Helpers]] (`asp-controller`, `asp-action`, `asp-for`, `asp-route-id`)
- [[Partial Views and View Components|Partial views]] for reusable card components
- [[Layouts and Sections|Sections]] (`@section Scripts`) for script placement

> [!summary] Section Summary
> - A real-world Razor view combines multiple syntax features into a cohesive page
> - Strong typing with `@model` provides IntelliSense and compile-time checking
> - Conditional rendering, iteration, and computed values live naturally alongside HTML
> - Tag helpers, partial views, and sections integrate seamlessly with Razor syntax

---
