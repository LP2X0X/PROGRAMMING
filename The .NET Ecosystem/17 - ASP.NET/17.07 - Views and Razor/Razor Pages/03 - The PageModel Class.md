---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


The **PageModel** class is the code-behind for a Razor Page. It is the equivalent of a controller action but scoped to a single page.

```csharp
// /Pages/Products/Index.cshtml.cs
public class IndexModel : PageModel
{
    private readonly IProductService _productService;

    public IndexModel(IProductService productService)
    {
        _productService = productService;
    }

    public List<Product> Products { get; set; } = new();

    public async Task OnGetAsync()
    {
        Products = await _productService.GetAllAsync();
    }
}
```

```cshtml
@* /Pages/Products/Index.cshtml *@
@page
@model MyApp.Pages.Products.IndexModel

<h1>Products</h1>
<table class="table">
    <thead>
        <tr>
            <th>Name</th>
            <th>Price</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var product in Model.Products)
        {
            <tr>
                <td>@product.Name</td>
                <td>@product.Price.ToString("C")</td>
                <td>
                    <a asp-page="./Edit" asp-route-id="@product.Id">Edit</a>
                    <a asp-page="./Delete" asp-route-id="@product.Id">Delete</a>
                </td>
            </tr>
        }
    </tbody>
</table>
<a asp-page="./Create" class="btn btn-primary">Create New Product</a>
```

**Key points:**
- The PageModel inherits from `Microsoft.AspNetCore.Mvc.RazorPages.PageModel`
- Constructor injection works just like controllers
- Public properties on the PageModel are accessible in the view via `Model.PropertyName`
- The `@model` directive in the `.cshtml` file points to the PageModel class (not a view model)

> [!warning] Common Misconception
> In MVC, `@model` refers to a view model (a data class). In Razor Pages, `@model` refers to the **PageModel class** (the code-behind). The PageModel is both the handler (like a controller) and the data provider (like a view model). Properties on the PageModel serve as the view model.

> [!summary] Section Summary
> - The PageModel class (`*.cshtml.cs`) is the code-behind that handles requests and holds page data
> - It supports constructor injection for services
> - Public properties on the PageModel are the view's data source (accessed via `Model.*`)
> - In Razor Pages, `@model` points to the PageModel, not a separate view model class

---
