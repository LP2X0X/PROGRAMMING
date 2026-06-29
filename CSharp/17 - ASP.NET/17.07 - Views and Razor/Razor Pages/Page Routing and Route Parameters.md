---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


The `@page` directive can include a route template for dynamic URL segments:

```cshtml
@* Simple required parameter *@
@page "{id:int}"
@* URL: /Products/Details/42 *@

@* Optional parameter *@
@page "{id:int?}"
@* URL: /Products/Details or /Products/Details/42 *@

@* Multiple parameters *@
@page "{category}/{id:int}"
@* URL: /Products/electronics/42 *@

@* Catch-all parameter *@
@page "{*slug}"
@* URL: /Blog/2024/01/my-post-title (slug = "2024/01/my-post-title") *@
```

### Accessing Route Parameters in the PageModel

Route parameters are passed as method parameters to handlers:

```csharp
public class DetailsModel : PageModel
{
    // Option 1: Handler method parameter
    public async Task OnGetAsync(int id)
    {
        Product = await _productService.GetByIdAsync(id);
    }

    // Option 2: [BindProperty] with SupportsGet
    [BindProperty(SupportsGet = true)]
    public int Id { get; set; }

    public async Task OnGetAsync()
    {
        Product = await _productService.GetByIdAsync(Id);
    }
}
```

### Route Constraints

Standard ASP.NET Core [[17.05 - Routing|route constraints]] work in Razor Pages:

| Constraint | Example | Matches |
|---|---|---|
| `int` | `{id:int}` | `42`, `1` |
| `bool` | `{active:bool}` | `true`, `false` |
| `datetime` | `{date:datetime}` | `2024-01-15` |
| `guid` | `{id:guid}` | `CD2C1638-...` |
| `minlength(n)` | `{slug:minlength(3)}` | Strings with 3+ chars |
| `regex(expr)` | `{code:regex(^[A-Z]{{3}}$)}` | Three uppercase letters |

### Overriding the Route Entirely

```cshtml
@page "/custom/url/path/{id:int}"
@* This page is now at /custom/url/path/42, not its file-system path *@
```

> [!summary] Section Summary
> - `@page "{id:int}"` adds route parameters to the page's URL
> - Parameters can be optional (`?`), constrained (`int`, `guid`, `minlength`), or catch-all (`*`)
> - Route parameters are accessible as handler method parameters or `[BindProperty(SupportsGet = true)]`
> - The entire route can be overridden: `@page "/custom/path/{id}"`

---
