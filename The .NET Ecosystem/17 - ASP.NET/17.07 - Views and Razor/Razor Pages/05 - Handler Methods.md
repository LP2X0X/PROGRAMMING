---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


Handler methods in a PageModel respond to HTTP verbs. They follow the naming convention `On{Verb}[Async]`:

### OnGet / OnGetAsync

Handles GET requests -- typically loads data for display:

```csharp
public async Task OnGetAsync()
{
    Products = await _productService.GetAllAsync();
}
```

### OnPost / OnPostAsync

Handles POST requests -- typically processes form submissions:

```csharp
public async Task<IActionResult> OnPostAsync()
{
    if (!ModelState.IsValid)
    {
        return Page();  // Re-render the page with validation errors
    }

    await _productService.CreateAsync(Product);
    return RedirectToPage("./Index");
}
```

### Return Types

Handler methods can return:

| Return Type | Use Case |
|---|---|
| `void` / `Task` | Render the current page (implicit `Page()`) |
| `IActionResult` / `Task<IActionResult>` | Return `Page()`, `RedirectToPage()`, `NotFound()`, etc. |
| `PageResult` | Explicitly render the current page |

```csharp
// Implicit return -- renders the page
public async Task OnGetAsync()
{
    Products = await _productService.GetAllAsync();
}

// Explicit return -- can redirect, return 404, etc.
public async Task<IActionResult> OnGetAsync(int id)
{
    Product = await _productService.GetByIdAsync(id);
    if (Product == null)
    {
        return NotFound();
    }
    return Page();
}
```

> [!tip] Practical Tip
> Use `void`/`Task` return types for simple GET handlers that always render the page. Use `IActionResult`/`Task<IActionResult>` when the handler might redirect, return a 404, or return different results based on conditions.

> [!summary] Section Summary
> - Handler methods follow the `On{Verb}[Async]` naming convention
> - `OnGetAsync()` handles GET requests (data loading)
> - `OnPostAsync()` handles POST requests (form processing)
> - Return `Page()` to render, `RedirectToPage()` to navigate, `NotFound()` for 404s
> - `void`/`Task` return types implicitly call `Page()`

---
