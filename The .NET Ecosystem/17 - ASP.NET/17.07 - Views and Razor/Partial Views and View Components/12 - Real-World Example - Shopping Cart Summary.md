---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


A complete view component that displays a cart summary in the site's navigation bar, independently fetching cart data:

### The Service Interface

```csharp
public interface ICartService
{
    Task<Cart> GetCartAsync(string sessionId);
}

public class Cart
{
    public List<CartItem> Items { get; set; } = new();
    public int TotalItems => Items.Sum(i => i.Quantity);
    public decimal TotalPrice => Items.Sum(i => i.Price * i.Quantity);
}

public class CartItem
{
    public int ProductId { get; set; }
    public string ProductName { get; set; }
    public decimal Price { get; set; }
    public int Quantity { get; set; }
}
```

### The View Component Class

```csharp
// /ViewComponents/CartSummaryViewComponent.cs
public class CartSummaryViewComponent : ViewComponent
{
    private readonly ICartService _cartService;

    public CartSummaryViewComponent(ICartService cartService)
    {
        _cartService = cartService;
    }

    public async Task<IViewComponentResult> InvokeAsync()
    {
        var sessionId = HttpContext.Session.GetString("CartSessionId") ?? string.Empty;

        if (string.IsNullOrEmpty(sessionId))
        {
            return View("Empty");
        }

        var cart = await _cartService.GetCartAsync(sessionId);
        return View(cart);
    }
}
```

### The Default View

```cshtml
@* /Views/Shared/Components/CartSummary/Default.cshtml *@
@model Cart

<a asp-controller="Cart" asp-action="Index" class="btn btn-outline-light position-relative">
    Cart
    @if (Model.TotalItems > 0)
    {
        <span class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-danger">
            @Model.TotalItems
        </span>
    }
</a>
```

### The Empty View

```cshtml
@* /Views/Shared/Components/CartSummary/Empty.cshtml *@
<a asp-controller="Cart" asp-action="Index" class="btn btn-outline-secondary">
    Cart (Empty)
</a>
```

### Invoking from the Layout

```cshtml
@* In _Layout.cshtml, inside the navbar *@
<div class="navbar-nav">
    <vc:cart-summary />
</div>
```

The layout has no knowledge of carts, cart services, or session IDs. The view component encapsulates all of that logic and renders the appropriate HTML fragment independently.

> [!summary] Section Summary
> - The cart summary view component independently fetches cart data from a service
> - It uses named views (`Default.cshtml` vs `Empty.cshtml`) for different states
> - The layout invokes it with `<vc:cart-summary />` without knowing anything about carts
> - Constructor injection provides access to `ICartService` for data fetching
> - This pattern demonstrates clean separation of concerns

---
