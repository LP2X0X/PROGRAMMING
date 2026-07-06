---
tags:
  - csharp
  - asp-net-core
  - routing
---


Route values are the **key-value pairs extracted from the URL** when a route template matches an incoming request. They are the bridge between the raw URL string and the strongly-typed parameters your code works with.

### How Route Values Are Produced

Given this route template and URL:

```
Template:  /products/{id}/reviews/{reviewId}
URL:       /products/42/reviews/7
```

The routing middleware produces these route values:

| Key | Value |
|---|---|
| `id` | `"42"` |
| `reviewId` | `"7"` |

All route values are initially **strings**. Conversion to `int`, `Guid`, etc. happens later during [[02 - Binding Sources (Default Priority Order)|model binding]].

### Accessing Route Values

Route values are stored in `HttpContext.Request.RouteValues`, a `RouteValueDictionary`:

```csharp
// In a controller action
public IActionResult GetProduct(int id)
{
    // Model binding already converted the route value to int
    // But you can also access the raw dictionary:
    var rawId = HttpContext.Request.RouteValues["id"]; // "42" as object
}
```

```csharp
// In middleware (after UseRouting)
app.Use(async (context, next) =>
{
    var id = context.Request.RouteValues["id"];
    await next();
});
```

See [[06 - Accessing Endpoint Info in Middleware]] for more on reading route data from middleware.

### Route Values from Defaults

Route values don't always come from the URL. **Default values** in the template also produce route values:

```csharp
// Conventional route with defaults
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

| URL | controller | action | id |
|---|---|---|---|
| `/` | `"Home"` | `"Index"` | `null` |
| `/Products` | `"Products"` | `"Index"` | `null` |
| `/Products/Details/5` | `"Products"` | `"Details"` | `"5"` |

The first row has route values entirely from defaults -- nothing was extracted from the URL.

### Route Values in URL Generation

Route values work in reverse for **link generation**. You supply route values, and the routing system produces a URL:

```csharp
// In a controller
var url = Url.Action("Details", "Products", new { id = 42 });
// Produces: /Products/Details/42
```

```csharp
// In Razor
<a asp-controller="Products" asp-action="Details" asp-route-id="42">View</a>
```

See [[06 - Route Names and URL Generation]] for the full URL generation story.

### Route Values vs Other Binding Sources

Route values are just one of several places [[02 - Binding Sources (Default Priority Order)|model binding]] looks for data:

| Source | Example | Priority (MVC) |
|---|---|---|
| **Route values** | `/products/{id}` | 1st |
| Query string | `?id=42` | 2nd |
| Form body | `<input name="id">` | 3rd |

For `[ApiController]`, the priority changes -- see [[02 - Binding Sources (Default Priority Order)]].

### Route Values vs Route Parameters vs Route Constraints

These three terms are related but distinct:

- **Route parameter** -- the `{id}` placeholder in the template (see [[05 - Route Parameters]])
- **Route value** -- the actual `"42"` extracted at runtime when a URL matches
- **Route constraint** -- a rule like `:int` that restricts which values match (see [[01 - What Are Route Constraints]])

> [!summary] Section Summary
> Route values are the key-value pairs produced when the routing middleware matches a URL against a route template. They are stored in `HttpContext.Request.RouteValues`, used by model binding to populate action parameters, and used in reverse for URL generation. Values come from URL segments, template defaults, or both.
