---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


The **anchor tag helper** generates correct URLs from routing information, eliminating hardcoded paths:

```cshtml
@* Basic usage *@
<a asp-controller="Products" asp-action="Index">All Products</a>
@* Renders: <a href="/Products">All Products</a> *@

@* With route parameter *@
<a asp-controller="Products" asp-action="Details" asp-route-id="42">
    Widget Pro
</a>
@* Renders: <a href="/Products/Details/42">Widget Pro</a> *@

@* Multiple route parameters *@
<a asp-controller="Products" asp-action="ByCategory"
   asp-route-category="electronics"
   asp-route-page="2">
    Electronics - Page 2
</a>
@* Renders: <a href="/Products/ByCategory/electronics?page=2">Electronics - Page 2</a> *@

@* Using a named route *@
<a asp-route="product-detail" asp-route-id="42">Widget Pro</a>

@* Linking to an area *@
<a asp-area="Admin" asp-controller="Dashboard" asp-action="Index">Admin</a>

@* Linking to a Razor Page *@
<a asp-page="/Products/Details" asp-route-id="42">Widget Pro</a>

@* Fragment (hash) *@
<a asp-controller="Products" asp-action="Details"
   asp-route-id="42" asp-fragment="reviews">
    See Reviews
</a>
@* Renders: <a href="/Products/Details/42#reviews">See Reviews</a> *@
```

**Available attributes:**

| Attribute | Purpose |
|---|---|
| `asp-controller` | Target controller name |
| `asp-action` | Target action method name |
| `asp-route-{name}` | Route parameter value (any name after the dash) |
| `asp-route` | Named route |
| `asp-area` | MVC area name |
| `asp-page` | Target Razor Page |
| `asp-page-handler` | Razor Page handler name |
| `asp-protocol` | Force `https` or `http` |
| `asp-host` | Override the hostname |
| `asp-fragment` | URL fragment (`#section`) |

> [!warning] Common Misconception
> `asp-route-id` is not a special attribute. The `id` part is just the route parameter name. You can use `asp-route-anything` to pass any route value: `asp-route-category`, `asp-route-slug`, `asp-route-year`, etc.

> [!summary] Section Summary
> - The anchor tag helper replaces `@Html.ActionLink()` with natural HTML anchor elements
> - `asp-controller`, `asp-action`, and `asp-route-{param}` generate correct URLs from routing
> - Works with areas, Razor Pages, named routes, fragments, and protocol overrides
> - `asp-route-{name}` is a dynamic attribute where `{name}` is any route parameter

---
