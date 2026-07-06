---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


In Razor Pages, model binding uses the `[BindProperty]` attribute to bind form data to PageModel properties:

```csharp
public class CreateModel : PageModel
{
    private readonly IProductService _productService;

    public CreateModel(IProductService productService)
    {
        _productService = productService;
    }

    [BindProperty]
    public Product Product { get; set; }

    public void OnGet()
    {
        // Product is not bound on GET -- initialize defaults if needed
        Product = new Product { Price = 0 };
    }

    public async Task<IActionResult> OnPostAsync()
    {
        // Product is automatically populated from form data
        if (!ModelState.IsValid)
        {
            return Page();
        }

        await _productService.CreateAsync(Product);
        return RedirectToPage("./Index");
    }
}
```

### Key Behaviors

- `[BindProperty]` binds on **POST by default** (not GET)
- To bind on GET as well: `[BindProperty(SupportsGet = true)]`
- Multiple properties can be bound:

```csharp
[BindProperty]
public Product Product { get; set; }

[BindProperty]
public List<int> SelectedCategoryIds { get; set; }
```

### Binding All Properties

```csharp
[BindProperties]  // Note the plural -- binds ALL public properties
public class CreateModel : PageModel
{
    public Product Product { get; set; }
    public string Notes { get; set; }

    // Both are bound from form data on POST
}
```

> [!warning] Common Misconception
> Properties without `[BindProperty]` are NOT populated from form data. If you forget the attribute, the property will be `null` in `OnPostAsync()` even if the form sends the data. This is the most common Razor Pages debugging issue for beginners.

> [!tip] Practical Tip
> Be selective with `[BindProperty]`. Only bind properties that come from the form. Properties that hold display-only data (like dropdown lists) should NOT be bound -- they need to be repopulated in the handler if `Page()` is returned:
> ```csharp
> [BindProperty]
> public Product Product { get; set; }      // From form
>
> public SelectList Categories { get; set; } // Display only -- no [BindProperty]
>
> public async Task<IActionResult> OnPostAsync()
> {
>     if (!ModelState.IsValid)
>     {
>         Categories = await LoadCategories(); // Must repopulate!
>         return Page();
>     }
>     // ...
> }
> ```

> [!summary] Section Summary
> - `[BindProperty]` marks properties for automatic model binding from form data
> - By default, binding only occurs on POST; use `SupportsGet = true` for GET binding
> - `[BindProperties]` (plural) binds all public properties on the PageModel
> - Forgetting `[BindProperty]` results in null properties -- the most common Razor Pages mistake
> - Display-only properties (dropdowns, lists) must be repopulated when returning `Page()`

---
