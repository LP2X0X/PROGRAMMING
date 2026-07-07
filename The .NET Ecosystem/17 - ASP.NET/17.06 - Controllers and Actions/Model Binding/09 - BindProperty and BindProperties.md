---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


While action method parameters are bound automatically, **controller properties** and **Razor Pages PageModel properties** are not bound by default. The `[BindProperty]` and `[BindProperties]` attributes opt in to property-level binding.

### [BindProperty] on Individual Properties

```csharp
public class ProductsController : Controller
{
    [BindProperty]
    public string SearchTerm { get; set; } = string.Empty;
    
    [BindProperty]
    public int Page { get; set; } = 1;
    
    [HttpPost]
    public IActionResult Search()
    {
        // SearchTerm and Page are bound from the request on POST
        var results = _service.Search(SearchTerm, Page);
        return View(results);
    }
}
```

### [BindProperties] on the Class

```csharp
[BindProperties]
public class ProductsController : Controller
{
    public string SearchTerm { get; set; } = string.Empty;
    public int Page { get; set; } = 1;
    public string SortBy { get; set; } = "name";
    
    // ALL public properties are bound from the request
}
```

### GET Requests and SupportsGet

By default, `[BindProperty]` and `[BindProperties]` only bind on **POST, PUT, PATCH, and DELETE** requests -- not on GET. To enable binding on GET requests, set `SupportsGet = true`:

```csharp
public class ProductsController : Controller
{
    [BindProperty(SupportsGet = true)]
    public string? SearchTerm { get; set; }
    
    [BindProperty(SupportsGet = true)]
    public int Page { get; set; } = 1;
    
    [HttpGet]
    public IActionResult Search()
    {
        // GET /products?searchTerm=laptop&page=2
        // SearchTerm and Page are now bound from query string on GET
    }
}
```

### Razor Pages PageModel Example

`[BindProperty]` is especially common in Razor Pages, where the PageModel serves as both the handler and the view model:

```csharp
public class CreateProductModel : PageModel
{
    private readonly IProductService _productService;
    
    public CreateProductModel(IProductService productService)
    {
        _productService = productService;
    }
    
    [BindProperty]
    public ProductInputModel Input { get; set; } = new();
    
    // Display data -- not bound from request (no [BindProperty])
    public List<string> Categories { get; set; } = new();
    
    public void OnGet()
    {
        Categories = _productService.GetCategories();
    }
    
    public async Task<IActionResult> OnPostAsync()
    {
        // Input is automatically bound from the form on POST
        if (!ModelState.IsValid)
        {
            Categories = _productService.GetCategories();
            return Page();
        }
        
        await _productService.CreateAsync(Input);
        return RedirectToPage("./Index");
    }
}

public class ProductInputModel
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; } = string.Empty;
    
    [Range(0.01, 99999.99)]
    public decimal Price { get; set; }
    
    public string? Description { get; set; }
    public string? Category { get; set; }
}
```

```ad-attention
Be cautious with `[BindProperties]` on controller classes -- it binds **all** public properties, which could expose properties you did not intend to be settable from request data. Prefer `[BindProperty]` on individual properties for tighter control.
```
