---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


A complete set of CRUD pages for managing products.

### Index Page (List)

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

    [BindProperty(SupportsGet = true)]
    public string SearchTerm { get; set; }

    public async Task OnGetAsync()
    {
        Products = string.IsNullOrWhiteSpace(SearchTerm)
            ? await _productService.GetAllAsync()
            : await _productService.SearchAsync(SearchTerm);
    }
}
```

```cshtml
@* /Pages/Products/Index.cshtml *@
@page
@model MyApp.Pages.Products.IndexModel

<h1>Products</h1>

<form method="get" class="mb-3">
    <div class="input-group">
        <input asp-for="SearchTerm" class="form-control" placeholder="Search products..." />
        <button type="submit" class="btn btn-outline-secondary">Search</button>
    </div>
</form>

<a asp-page="./Create" class="btn btn-primary mb-3">Create New Product</a>

@if (!Model.Products.Any())
{
    <p class="text-muted">No products found.</p>
}
else
{
    <table class="table table-striped">
        <thead>
            <tr>
                <th>Name</th>
                <th>Category</th>
                <th>Price</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var product in Model.Products)
            {
                <tr>
                    <td>@product.Name</td>
                    <td>@product.CategoryName</td>
                    <td>@product.Price.ToString("C")</td>
                    <td>
                        <a asp-page="./Edit" asp-route-id="@product.Id"
                           class="btn btn-sm btn-outline-primary">Edit</a>
                        <a asp-page="./Delete" asp-route-id="@product.Id"
                           class="btn btn-sm btn-outline-danger">Delete</a>
                    </td>
                </tr>
            }
        </tbody>
    </table>
}
```

### Create Page

```csharp
// /Pages/Products/Create.cshtml.cs
public class CreateModel : PageModel
{
    private readonly IProductService _productService;
    private readonly ICategoryService _categoryService;

    public CreateModel(IProductService productService, ICategoryService categoryService)
    {
        _productService = productService;
        _categoryService = categoryService;
    }

    [BindProperty]
    public ProductCreateDto Product { get; set; }

    public SelectList Categories { get; set; }

    public async Task OnGetAsync()
    {
        Categories = new SelectList(
            await _categoryService.GetAllAsync(), "Id", "Name");
    }

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid)
        {
            // Must repopulate non-bound properties
            Categories = new SelectList(
                await _categoryService.GetAllAsync(), "Id", "Name");
            return Page();
        }

        await _productService.CreateAsync(Product);
        TempData["SuccessMessage"] = "Product created successfully.";
        return RedirectToPage("./Index");
    }
}
```

```cshtml
@* /Pages/Products/Create.cshtml *@
@page
@model MyApp.Pages.Products.CreateModel

<h1>Create Product</h1>

<form method="post">
    <div asp-validation-summary="All" class="text-danger"></div>

    <div class="mb-3">
        <label asp-for="Product.Name"></label>
        <input asp-for="Product.Name" class="form-control" />
        <span asp-validation-for="Product.Name" class="text-danger"></span>
    </div>

    <div class="mb-3">
        <label asp-for="Product.Description"></label>
        <textarea asp-for="Product.Description" class="form-control" rows="4"></textarea>
        <span asp-validation-for="Product.Description" class="text-danger"></span>
    </div>

    <div class="mb-3">
        <label asp-for="Product.Price"></label>
        <input asp-for="Product.Price" class="form-control" />
        <span asp-validation-for="Product.Price" class="text-danger"></span>
    </div>

    <div class="mb-3">
        <label asp-for="Product.CategoryId"></label>
        <select asp-for="Product.CategoryId" asp-items="Model.Categories" class="form-select">
            <option value="">-- Select Category --</option>
        </select>
        <span asp-validation-for="Product.CategoryId" class="text-danger"></span>
    </div>

    <button type="submit" class="btn btn-primary">Create</button>
    <a asp-page="./Index" class="btn btn-secondary">Cancel</a>
</form>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

### Edit Page

```csharp
// /Pages/Products/Edit.cshtml.cs
public class EditModel : PageModel
{
    private readonly IProductService _productService;
    private readonly ICategoryService _categoryService;

    public EditModel(IProductService productService, ICategoryService categoryService)
    {
        _productService = productService;
        _categoryService = categoryService;
    }

    [BindProperty]
    public ProductEditDto Product { get; set; }

    public SelectList Categories { get; set; }

    public async Task<IActionResult> OnGetAsync(int id)
    {
        var product = await _productService.GetByIdAsync(id);
        if (product == null)
        {
            return NotFound();
        }

        Product = new ProductEditDto
        {
            Id = product.Id,
            Name = product.Name,
            Description = product.Description,
            Price = product.Price,
            CategoryId = product.CategoryId
        };

        Categories = new SelectList(
            await _categoryService.GetAllAsync(), "Id", "Name");

        return Page();
    }

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid)
        {
            Categories = new SelectList(
                await _categoryService.GetAllAsync(), "Id", "Name");
            return Page();
        }

        var success = await _productService.UpdateAsync(Product);
        if (!success)
        {
            return NotFound();
        }

        TempData["SuccessMessage"] = "Product updated successfully.";
        return RedirectToPage("./Index");
    }
}
```

### Delete Page (Confirmation)

```csharp
// /Pages/Products/Delete.cshtml.cs
public class DeleteModel : PageModel
{
    private readonly IProductService _productService;

    public DeleteModel(IProductService productService)
    {
        _productService = productService;
    }

    public Product Product { get; set; }

    public async Task<IActionResult> OnGetAsync(int id)
    {
        Product = await _productService.GetByIdAsync(id);
        if (Product == null)
        {
            return NotFound();
        }
        return Page();
    }

    public async Task<IActionResult> OnPostAsync(int id)
    {
        var success = await _productService.DeleteAsync(id);
        if (!success)
        {
            return NotFound();
        }

        TempData["SuccessMessage"] = "Product deleted.";
        return RedirectToPage("./Index");
    }
}
```

```cshtml
@* /Pages/Products/Delete.cshtml *@
@page "{id:int}"
@model MyApp.Pages.Products.DeleteModel

<h1>Delete Product</h1>

<div class="alert alert-danger">
    <h4 class="alert-heading">Are you sure?</h4>
    <p>You are about to permanently delete the following product:</p>
</div>

<dl class="row">
    <dt class="col-sm-3">Name</dt>
    <dd class="col-sm-9">@Model.Product.Name</dd>

    <dt class="col-sm-3">Category</dt>
    <dd class="col-sm-9">@Model.Product.CategoryName</dd>

    <dt class="col-sm-3">Price</dt>
    <dd class="col-sm-9">@Model.Product.Price.ToString("C")</dd>
</dl>

<form method="post" asp-route-id="@Model.Product.Id">
    <button type="submit" class="btn btn-danger">Confirm Delete</button>
    <a asp-page="./Index" class="btn btn-secondary">Cancel</a>
</form>
```

This CRUD set demonstrates:
- Handler methods for GET and POST
- `[BindProperty]` for form model binding
- Route parameters (`{id:int}`)
- `RedirectToPage()` for post-redirect-get (PRG) pattern
- Validation with `ModelState.IsValid` and `Page()` for re-rendering
- Repopulating non-bound properties (Categories) when returning `Page()`
- [[Tag Helpers]] for forms, links, and validation
- `TempData` for success messages across redirects

> [!summary] Section Summary
> - A complete CRUD set consists of Index, Create, Edit, and Delete pages
> - Each page follows the PageModel pattern with GET (display) and POST (process) handlers
> - The post-redirect-get (PRG) pattern prevents duplicate form submissions
> - Non-bound display data must be repopulated when returning `Page()` after validation failure
> - `TempData` carries success messages across the redirect

---
