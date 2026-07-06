---
tags: [csharp, asp-net-core, project-structure]
---


The `Pages/` folder is used in Razor Pages projects (`dotnet new webapp`). Unlike MVC, Razor Pages couples the view and its handler logic into a page-focused model.

### Folder Structure

```
Pages/
  Shared/
    _Layout.cshtml
    _Layout.cshtml.css
    _ValidationScriptsPartial.cshtml
  _ViewImports.cshtml
  _ViewStart.cshtml
  Index.cshtml
  Index.cshtml.cs
  Privacy.cshtml
  Privacy.cshtml.cs
  Error.cshtml
  Error.cshtml.cs
  Orders/
    Index.cshtml
    Index.cshtml.cs
    Details.cshtml
    Details.cshtml.cs
```

### Page Model Example

Each `.cshtml` page has a paired `.cshtml.cs` code-behind file (the PageModel):

```csharp
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace OrderManagement.Pages.Orders;

public class DetailsModel : PageModel
{
    private readonly IOrderService _orderService;

    public DetailsModel(IOrderService orderService)
    {
        _orderService = orderService;
    }

    public Order Order { get; set; } = default!;

    public async Task<IActionResult> OnGetAsync(int id)
    {
        var order = await _orderService.GetByIdAsync(id);
        if (order is null)
            return NotFound();

        Order = order;
        return Page();
    }
}
```

### Razor Pages Routing

Razor Pages use file-based routing. The URL maps directly to the file path under `Pages/`:

| File Path | URL |
|---|---|
| `Pages/Index.cshtml` | `/` or `/Index` |
| `Pages/Privacy.cshtml` | `/Privacy` |
| `Pages/Orders/Index.cshtml` | `/Orders` or `/Orders/Index` |
| `Pages/Orders/Details.cshtml` | `/Orders/Details?id=5` |

> [!ad-note] MVC vs Razor Pages
> MVC separates concerns into Controllers, Models, and Views across different folders. Razor Pages keeps the view and its handler together in one location. Razor Pages is recommended for page-focused scenarios (forms, simple CRUD), while MVC is better for complex applications with shared logic across multiple views.

> [!summary] Section Summary
> - `Pages/` is the Razor Pages equivalent of `Controllers/` + `Views/`
> - Each page consists of a `.cshtml` view and a `.cshtml.cs` PageModel code-behind
> - Routing is file-based: the folder/file structure determines the URL
> - Razor Pages groups the view and its logic together, simplifying page-focused development
