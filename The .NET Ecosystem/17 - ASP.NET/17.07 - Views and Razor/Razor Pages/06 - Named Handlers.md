---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


When a page needs multiple POST actions (e.g., an edit page with both "Save" and "Delete" buttons), use **named handlers**:

```csharp
// /Pages/Products/Edit.cshtml.cs
public class EditModel : PageModel
{
    // ...

    public async Task<IActionResult> OnPostSaveAsync()
    {
        if (!ModelState.IsValid)
        {
            return Page();
        }
        await _productService.UpdateAsync(Product);
        return RedirectToPage("./Index");
    }

    public async Task<IActionResult> OnPostDeleteAsync()
    {
        await _productService.DeleteAsync(Product.Id);
        return RedirectToPage("./Index");
    }

    public async Task<IActionResult> OnPostPublishAsync()
    {
        await _productService.PublishAsync(Product.Id);
        return RedirectToPage("./Edit", new { id = Product.Id });
    }
}
```

```cshtml
@* /Pages/Products/Edit.cshtml *@
@page "{id:int}"
@model EditModel

<form method="post">
    <input asp-for="Product.Id" type="hidden" />
    <div class="mb-3">
        <label asp-for="Product.Name"></label>
        <input asp-for="Product.Name" class="form-control" />
        <span asp-validation-for="Product.Name" class="text-danger"></span>
    </div>
    <div class="mb-3">
        <label asp-for="Product.Price"></label>
        <input asp-for="Product.Price" class="form-control" />
        <span asp-validation-for="Product.Price" class="text-danger"></span>
    </div>

    <button type="submit" asp-page-handler="Save" class="btn btn-primary">Save</button>
    <button type="submit" asp-page-handler="Publish" class="btn btn-success">Publish</button>
    <button type="submit" asp-page-handler="Delete" class="btn btn-danger"
            onclick="return confirm('Are you sure?')">
        Delete
    </button>
</form>
```

**How it works:**
- `asp-page-handler="Save"` adds `?handler=Save` to the form's action URL
- ASP.NET Core matches the `handler` query parameter to `OnPost**Save**Async()`
- The handler name is the part between `OnPost` and `Async`

> [!ad-note] URL Format
> Named handlers append `?handler=Name` to the URL by default. If you prefer clean URLs, you can configure route-based handler names:
> ```csharp
> builder.Services.AddRazorPages().AddRazorPagesOptions(options =>
> {
>     options.Conventions.ConfigureFilter(new AutoValidateAntiforgeryTokenAttribute());
> });
> ```
> Or use a route template: `@page "{id:int}/{handler?}"`

> [!summary] Section Summary
> - Named handlers allow multiple POST actions on a single page (`OnPostSaveAsync`, `OnPostDeleteAsync`)
> - `asp-page-handler="Name"` on buttons/forms routes to the corresponding handler
> - The handler name maps to the method name: `asp-page-handler="Save"` -> `OnPostSaveAsync()`
> - By default, handlers are passed via `?handler=Name` query parameter

---
