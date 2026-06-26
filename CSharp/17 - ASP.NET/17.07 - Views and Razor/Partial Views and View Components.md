---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---

# Partial Views and View Components

> [!ad-note] Overview
> ASP.NET Core provides two mechanisms for creating reusable view fragments: **partial views** for simple, model-driven HTML snippets, and **view components** for self-contained mini-controllers that fetch their own data. This note covers when to use each, how they work internally, and practical patterns for building maintainable UIs.

## Table of Contents

- [Partial Views](#partial-views)
- [Rendering Partial Views](#rendering-partial-views)
- [Partial View Discovery and Naming](#partial-view-discovery-and-naming)
- [Passing Data to Partial Views](#passing-data-to-partial-views)
- [When Partial Views Are Not Enough](#when-partial-views-are-not-enough)
- [View Components](#view-components)
- [Creating a View Component](#creating-a-view-component)
- [The View Component View](#the-view-component-view)
- [Invoking View Components](#invoking-view-components)
- [Dependency Injection in View Components](#dependency-injection-in-view-components)
- [Partial Views vs View Components Comparison](#partial-views-vs-view-components-comparison)
- [Real-World Example: Shopping Cart Summary](#real-world-example-shopping-cart-summary)
- [Comprehensive Summary](#comprehensive-summary)
- [Related Topics](#related-topics)

---

## Partial Views

A **partial view** is a `.cshtml` file that renders a fragment of HTML. It is not a standalone page -- it is meant to be embedded within another view or [[Layouts and Sections|layout]]. Think of partials as reusable HTML components that receive their data from the parent view.

Common uses:
- Product cards in a product listing
- Navigation menus
- Form fragments shared between Create and Edit views
- Comment blocks in a blog
- Table rows with complex formatting

```cshtml
@* /Views/Shared/_ProductCard.cshtml *@
@model Product

<div class="card h-100">
    <img src="@Model.ImageUrl" class="card-img-top" alt="@Model.Name" />
    <div class="card-body">
        <h5 class="card-title">@Model.Name</h5>
        <p class="card-text text-muted">@Model.Category.Name</p>
        <p class="card-text fw-bold">@Model.Price.ToString("C")</p>
    </div>
    <div class="card-footer">
        <a asp-controller="Products" asp-action="Details" asp-route-id="@Model.Id"
           class="btn btn-primary btn-sm">
            View Details
        </a>
    </div>
</div>
```

> [!ad-note] Key Principle
> A partial view is purely a rendering concern. It takes data that has already been prepared and turns it into HTML. It does NOT fetch data from a database, call services, or perform business logic. If you need that, use a [[#View Components|view component]].

> [!summary] Section Summary
> - Partial views are reusable `.cshtml` fragments embedded within other views
> - They are presentation-only: they render data, they do not fetch it
> - Common for cards, form fragments, navigation menus, and repeated UI patterns
> - Named with an underscore prefix by convention (`_ProductCard.cshtml`)

---

## Rendering Partial Views

There are several ways to render a partial view. The **partial tag helper** is the modern, recommended approach.

### Partial Tag Helper (Recommended)

```cshtml
<partial name="_ProductCard" model="product" />
```

### Html.PartialAsync (Older Approach)

```cshtml
@await Html.PartialAsync("_ProductCard", product)
```

### Html.RenderPartialAsync (Writes Directly to Response)

```cshtml
@{ await Html.RenderPartialAsync("_ProductCard", product); }
```

> [!tip] Practical Tip
> Prefer the `<partial>` tag helper for clarity and consistency with the rest of the [[Tag Helpers]] ecosystem. `Html.PartialAsync` returns an `IHtmlContent` (buffered in memory), while `Html.RenderPartialAsync` writes directly to the output stream. For most cases the performance difference is negligible, but `RenderPartialAsync` can be slightly more efficient for very large partials.

### Rendering in a Loop

```cshtml
<div class="row">
    @foreach (var product in Model.Products)
    {
        <div class="col-md-4 mb-4">
            <partial name="_ProductCard" model="product" />
        </div>
    }
</div>
```

> [!warning] Common Misconception
> `Html.Partial()` (synchronous, without `Async`) exists but can cause deadlocks in ASP.NET Core. Always use the async variants or the `<partial>` tag helper. The synchronous methods are obsolete and will generate compiler warnings.

> [!summary] Section Summary
> - `<partial name="..." model="..." />` is the recommended way to render partials
> - `Html.PartialAsync()` and `Html.RenderPartialAsync()` are the programmatic alternatives
> - Always use async methods -- synchronous `Html.Partial()` risks deadlocks
> - Partials work naturally inside loops for rendering collections

---

## Partial View Discovery and Naming

### Naming Convention

Partial views are prefixed with an underscore (`_`). This is a **convention**, not a requirement, but it serves two purposes:
1. Visually distinguishes partials from full views in the file explorer
2. Prevents partials from being returned directly by a controller (convention-based routing skips `_`-prefixed files)

### Discovery Order

When you reference `<partial name="_ProductCard" />`, Razor searches in this order:

1. **Same folder** as the calling view (e.g., `/Views/Products/`)
2. **`/Views/Shared/`**
3. **`/Pages/Shared/`** (if using [[Razor Pages]])

If you need a partial from a specific location, use a full path:

```cshtml
<partial name="/Views/Admin/Shared/_AdminWidget.cshtml" />
```

> [!ad-note] Area Partials
> In area-based applications, the search also includes the area's `Shared` folder:
> `/Areas/Admin/Views/Shared/_AdminWidget.cshtml`

> [!summary] Section Summary
> - Partial views use an underscore prefix by convention (`_PartialName.cshtml`)
> - Razor searches the current view's folder first, then `/Views/Shared/`, then `/Pages/Shared/`
> - Use full paths when the default discovery order does not find the right partial

---

## Passing Data to Partial Views

### Strongly-Typed Model (Recommended)

The partial declares a `@model` directive and receives a typed object:

```cshtml
@* _ProductCard.cshtml *@
@model Product

<h3>@Model.Name</h3>
<p>@Model.Price.ToString("C")</p>
```

```cshtml
@* Calling view *@
<partial name="_ProductCard" model="myProduct" />
```

### ViewData / ViewBag

Partial views inherit the parent view's `ViewData` dictionary. You can also pass additional `ViewData`:

```cshtml
<partial name="_ProductCard"
         model="myProduct"
         view-data="@(new ViewDataDictionary(ViewData) { { "ShowAddToCart", true } })" />
```

```cshtml
@* Inside _ProductCard.cshtml *@
@if ((bool?)ViewData["ShowAddToCart"] == true)
{
    <button class="btn btn-primary">Add to Cart</button>
}
```

> [!warning] Common Misconception
> Partial views do NOT inherit the parent's `@model` automatically. If the partial has `@model Product` and you do not pass a model, `Model` will be `null`. The partial's `Model` property is independent of the parent view's `Model`.

> [!tip] Practical Tip
> Prefer strongly-typed models over `ViewData` for partials. Create a dedicated view model if a partial needs both a domain object and display flags:
> ```csharp
> public class ProductCardViewModel
> {
>     public Product Product { get; set; }
>     public bool ShowAddToCart { get; set; }
>     public bool ShowPrice { get; set; }
> }
> ```

> [!summary] Section Summary
> - Pass data to partials using the `model` attribute on the `<partial>` tag helper
> - Partials do NOT automatically inherit the parent's `@model` -- always pass data explicitly
> - `ViewData` can be passed for additional flags, but prefer dedicated view models
> - Strongly-typed models provide IntelliSense and compile-time safety

---

## When Partial Views Are Not Enough

Partial views hit their limits when the HTML fragment needs its own data. Consider this scenario:

You want to display a "cart summary" widget in the navigation bar (defined in the [[Layouts and Sections|layout]]). The layout does not have cart data -- it does not even know about carts. You would need to:

1. Add cart data to every controller action's view model, OR
2. Use `ViewData`/`ViewBag` and set it in every controller, OR
3. Use an action filter to inject cart data globally

All of these options are clunky and violate separation of concerns. This is exactly the problem **view components** solve.

> [!summary] Section Summary
> - Partial views require the calling view to provide all data
> - When a UI fragment needs its own independent data fetching, partials become impractical
> - Injecting data via ViewBag/ViewData or action filters is a code smell for this scenario
> - View components solve this by encapsulating data access and rendering together

---

## View Components

A **view component** is a self-contained unit that combines:
1. A **C# class** (the "mini-controller") that fetches data and prepares a model
2. A **Razor view** that renders the HTML

View components are similar to partial views, but they can independently query databases, call services, and perform logic -- all without the parent view or controller knowing about it.

### When to Use View Components

- Navigation menus that need to load menu items from a database
- Shopping cart summaries that query the cart service
- "Recently viewed" or "recommended products" widgets
- Sidebar content that varies by user role
- Any reusable UI widget that needs its own data pipeline

> [!ad-note] View Components vs Partial Views
> The mental model: if the fragment only *displays* data (already available from the parent), use a partial view. If the fragment needs to *fetch* its own data, use a view component.

> [!summary] Section Summary
> - View components are mini-controllers that produce HTML fragments with their own data
> - They combine a C# class (data fetching) and a Razor view (rendering)
> - Use them when the UI fragment needs independent data access
> - They encapsulate both logic and presentation, unlike partials which are presentation-only

---

## Creating a View Component

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

## The View Component View

View component views follow a specific file location convention:

```
/Views/Shared/Components/[ComponentName]/Default.cshtml
```

Or, for controller-specific view components:

```
/Views/[Controller]/Components/[ComponentName]/Default.cshtml
```

For the `CartSummaryViewComponent`:

```cshtml
@* /Views/Shared/Components/CartSummary/Default.cshtml *@
@model CartSummaryViewModel

<div class="cart-summary">
    <a asp-controller="Cart" asp-action="Index" class="nav-link">
        <span class="cart-icon">Cart</span>
        @if (Model.ItemCount > 0)
        {
            <span class="badge bg-danger">@Model.ItemCount</span>
            <span class="cart-total">@Model.TotalPrice.ToString("C")</span>
        }
        else
        {
            <span class="text-muted">Empty</span>
        }
    </a>
</div>
```

**Naming the view:**
- `Default.cshtml` is the default view name
- You can return a named view: `return View("Mini", model);` looks for `Mini.cshtml` in the same folder
- This allows multiple display modes for the same view component

```csharp
public async Task<IViewComponentResult> InvokeAsync(string displayMode = "default")
{
    var model = await BuildModel();
    return displayMode switch
    {
        "mini" => View("Mini", model),
        "full" => View("Full", model),
        _ => View(model)  // Default.cshtml
    };
}
```

> [!summary] Section Summary
> - View component views live in `/Views/Shared/Components/[Name]/Default.cshtml`
> - `Default.cshtml` is the default name; named views allow multiple display modes
> - The view receives a strongly-typed model from `InvokeAsync()` via `View(model)`
> - View component views support all [[Razor Syntax]] features, tag helpers, and nested partials

---

## Invoking View Components

There are two ways to invoke a view component from a Razor view.

### Tag Helper Syntax (Recommended)

```cshtml
@* Simple invocation *@
<vc:cart-summary />

@* With parameters *@
<vc:recent-products max-items="5" />

@* With a string parameter *@
<vc:navigation-menu area="admin" />
```

The tag helper uses **kebab-case**: `CartSummary` becomes `<vc:cart-summary>`, and `maxItems` becomes `max-items`.

> [!ad-note] Enabling the vc: Tag Helper
> The `<vc:>` tag helper requires `@addTagHelper` in `_ViewImports.cshtml`:
> ```cshtml
> @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
> ```
> This is the same directive that enables built-in [[Tag Helpers]] and is almost always already present.

### Component.InvokeAsync Syntax

```cshtml
@await Component.InvokeAsync("CartSummary")

@await Component.InvokeAsync("RecentProducts", new { maxItems = 5 })
```

This uses the component name as a string, which means no compile-time checking. Prefer the tag helper approach.

> [!tip] Practical Tip
> Use `<vc:component-name />` in views for readability and compile-time safety. Use `Component.InvokeAsync()` only when you need to invoke a view component dynamically (e.g., the component name comes from configuration).

> [!summary] Section Summary
> - `<vc:component-name />` is the recommended tag helper syntax (kebab-case)
> - `Component.InvokeAsync("Name")` is the programmatic alternative (string-based)
> - Parameters map from PascalCase to kebab-case: `maxItems` becomes `max-items`
> - The `vc:` prefix requires `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers`

---

## Dependency Injection in View Components

View components support constructor injection, just like controllers. This is one of their primary advantages over partial views.

```csharp
public class RecentProductsViewComponent : ViewComponent
{
    private readonly IProductRepository _productRepo;
    private readonly IMemoryCache _cache;
    private readonly ILogger<RecentProductsViewComponent> _logger;

    public RecentProductsViewComponent(
        IProductRepository productRepo,
        IMemoryCache cache,
        ILogger<RecentProductsViewComponent> logger)
    {
        _productRepo = productRepo;
        _cache = cache;
        _logger = logger;
    }

    public async Task<IViewComponentResult> InvokeAsync(int count = 4)
    {
        var cacheKey = $"recent-products-{count}";

        if (!_cache.TryGetValue(cacheKey, out List<Product> products))
        {
            _logger.LogInformation("Cache miss for recent products");
            products = await _productRepo.GetRecentAsync(count);

            _cache.Set(cacheKey, products, TimeSpan.FromMinutes(5));
        }

        return View(products);
    }
}
```

Services must be registered in the DI container as usual:

```csharp
// Program.cs
builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddMemoryCache();
```

> [!ad-note] Testability
> View components are straightforward to unit test because their dependencies are injected. Mock the services, call `InvokeAsync()`, and assert the returned `IViewComponentResult`. This is a significant advantage over embedding service calls in partial views via `@inject`.

> [!summary] Section Summary
> - View components support full constructor injection like controllers
> - Register dependencies in `Program.cs` as usual
> - This enables caching, logging, database access, and any service interaction
> - Constructor injection makes view components easy to unit test with mocked dependencies

---

## Partial Views vs View Components Comparison

| Aspect | Partial View | View Component |
|---|---|---|
| **Data source** | Receives data from the parent view | Fetches its own data independently |
| **C# logic** | Minimal (formatting only) | Can contain business/data logic |
| **DI support** | Only via `@inject` (limited) | Full constructor injection |
| **File structure** | Single `.cshtml` file | C# class + `.cshtml` view |
| **Invocation** | `<partial name="..." />` | `<vc:name />` or `Component.InvokeAsync()` |
| **Testability** | Hard to unit test | Easy to unit test (injectable dependencies) |
| **Caching** | No built-in support | Can implement caching in `InvokeAsync()` |
| **Complexity** | Low | Medium |
| **Typical uses** | Cards, form fields, list items | Nav menus, cart widgets, recommendations |
| **Analogy** | A function that formats data | A mini-controller with its own view |

**Decision guide:**
1. Does the fragment need to fetch data the parent does not have? **View Component**.
2. Does the fragment just render data the parent already prepared? **Partial View**.
3. Does the fragment need caching, logging, or complex service calls? **View Component**.
4. Is the fragment a simple HTML template with model binding? **Partial View**.

> [!summary] Section Summary
> - Partial views are simple and lightweight -- use them for rendering pre-prepared data
> - View components are powerful and self-contained -- use them for independent data fetching
> - The decision hinges on whether the fragment needs its own data pipeline
> - View components are more testable due to constructor injection

---

## Real-World Example: Shopping Cart Summary

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

## Comprehensive Summary

> [!tip] Complete Summary
> ASP.NET Core offers two complementary mechanisms for reusable view fragments: **partial views** and **view components**.
>
> **Partial views** are simple `.cshtml` files that render HTML from data provided by the parent view. They are invoked with `<partial name="_Name" model="data" />`, searched in the current folder then `/Views/Shared/`, and follow the `_Prefix` naming convention. They are ideal for repeated UI patterns like product cards, form fields, and list items where all data is already available. They should contain only presentation logic.
>
> **View components** are self-contained units with a C# class (`ViewComponent` subclass) and a Razor view (`/Views/Shared/Components/[Name]/Default.cshtml`). The class implements `InvokeAsync()` to fetch data via injected services, then returns `View(model)`. They are invoked with the `<vc:name />` tag helper or `Component.InvokeAsync()`. View components are the right choice when a UI fragment needs its own data pipeline -- navigation menus, cart widgets, recommendation engines, and any widget that cannot rely on the parent view for its data.
>
> The decision is simple: if the fragment **displays** pre-prepared data, use a partial view. If the fragment **fetches** its own data, use a view component. View components additionally offer full DI support, testability through constructor injection, and the ability to implement caching and complex logic within the `InvokeAsync()` method.

---

## Related Topics

- [[Razor Syntax]] -- the Razor markup used within partials and view component views
- [[Layouts and Sections]] -- layouts often host view components in navigation and sidebars
- [[Tag Helpers]] -- the `<partial>` and `<vc:>` tag helpers for invoking partials and view components
- [[Razor Pages]] -- Razor Pages use the same partial view and view component mechanisms
- [[17.03 - Dependency Injection]] -- the DI system that provides services to view components
- [[17.06 - Controllers and Actions]] -- controllers prepare data for views, which then use partials
