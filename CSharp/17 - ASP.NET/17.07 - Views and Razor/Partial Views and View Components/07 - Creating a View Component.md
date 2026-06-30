---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


A view component class inherits from `ViewComponent` and implements an `InvokeAsync()` method (or `Invoke()` for synchronous operations):

```csharp
// /ViewComponents/CartSummaryViewComponent.cs
using Microsoft.AspNetCore.Mvc;

public class CartSummaryViewComponent : ViewComponent
{
    private readonly ICartService _cartService;

    public CartSummaryViewComponent(ICartService cartService)
    {
        _cartService = cartService;
    }

    public async Task<IViewComponentResult> InvokeAsync()
    {
        var items = await _cartService.GetCartItemsAsync(HttpContext);
        var model = new CartSummaryViewModel
        {
            ItemCount = items.Count,
            TotalPrice = items.Sum(i => i.Price * i.Quantity)
        };
        return View(model);
    }
}
```

**Key points:**
- The class name convention is `[Name]ViewComponent` -- the `ViewComponent` suffix is stripped when invoking
- Constructor injection works exactly like controllers (register services in `Program.cs`)
- `InvokeAsync()` returns `Task<IViewComponentResult>`
- `View()` renders the associated Razor view with a model (just like `Controller.View()`)
- You can pass parameters to `InvokeAsync()`:

```csharp
public async Task<IViewComponentResult> InvokeAsync(int maxItems = 5)
{
    var items = await _productService.GetRecentProductsAsync(maxItems);
    return View(items);
}
```

### View Component Properties

Inside a view component, you have access to:
- `HttpContext` -- the current HTTP context
- `Request` -- the current request
- `User` -- the current `ClaimsPrincipal`
- `ViewData` / `TempData` -- data dictionaries
- `ModelState` -- model state dictionary
- `RouteData` -- route information

> [!warning] Common Misconception
> View components are NOT controllers. They do not participate in model binding, they cannot return redirects or status codes, and they do not have action filters. They are purely for rendering HTML fragments.

> [!summary] Section Summary
> - View components inherit from `ViewComponent` and implement `InvokeAsync()`
> - Constructor injection provides access to services (database, cache, etc.)
> - `View(model)` renders the associated Razor view with a model
> - Parameters to `InvokeAsync()` allow configuration from the calling view
> - View components have access to `HttpContext`, `User`, and other request data

---
