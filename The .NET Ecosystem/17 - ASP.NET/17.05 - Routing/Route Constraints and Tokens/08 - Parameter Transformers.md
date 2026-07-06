---
tags:
  - csharp
  - asp-net-core
  - routing
  - constraints
---


**Parameter transformers** modify how token values and route parameter values appear in generated URLs. The most common use is converting PascalCase names to kebab-case or lowercase.

### Built-In: SlugifyParameterTransformer

ASP.NET Core does not ship a built-in `slugify` transformer, but the pattern is well-established:

```csharp
using System.Text.RegularExpressions;

public class SlugifyParameterTransformer : IOutboundParameterTransformer
{
    public string? TransformOutbound(object? value)
    {
        if (value is null) return null;

        // PascalCase to kebab-case: "ProductCategories" -> "product-categories"
        return Regex.Replace(
            value.ToString()!,
            "([a-z])([A-Z])",
            "$1-$2",
            RegexOptions.CultureInvariant,
            TimeSpan.FromMilliseconds(100))
            .ToLowerInvariant();
    }
}
```

### Registration

```csharp
builder.Services.AddRouting(options =>
{
    options.ConstraintMap["slugify"] = typeof(SlugifyParameterTransformer);
});
```

### Using with Conventional Routes

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller:slugify}/{action:slugify}/{id?}");
```

Now `ProductCategoriesController.GetAllItems()` maps to `/product-categories/get-all-items`.

### Using with Attribute Routes (Global Convention)

For attribute routing, apply a convention globally rather than per-route:

```csharp
builder.Services.AddControllers(options =>
{
    options.Conventions.Add(
        new RouteTokenTransformerConvention(new SlugifyParameterTransformer()));
});
```

This transforms all `[controller]` and `[action]` token replacements:

```csharp
[Route("api/[controller]")]    // Resolves to "api/product-categories"
public class ProductCategoriesController : ControllerBase
{
    [HttpGet("[action]")]      // Resolves to "api/product-categories/get-all"
    public IActionResult GetAll() => Ok();
}
```

### Simple Lowercase URLs

For just lowercasing (without kebab-case), use the built-in option:

```csharp
builder.Services.AddRouting(options =>
{
    options.LowercaseUrls = true;           // /api/products instead of /api/Products
    options.LowercaseQueryStrings = true;   // ?category=electronics
    options.AppendTrailingSlash = false;     // No trailing slash
});
```

> [!ad-note] Key Insight
> `LowercaseUrls` affects **URL generation** (when you create URLs via `Url.Action()` or `LinkGenerator`), not **URL matching**. Route matching in ASP.NET Core is always case-insensitive by default. The setting ensures that generated URLs look clean and consistent.

> [!summary] Section Summary
> - Parameter transformers modify how token values appear in URLs (e.g., PascalCase to kebab-case).
> - Implement `IOutboundParameterTransformer` and register in `ConstraintMap`.
> - For attribute routing, use `RouteTokenTransformerConvention` to apply globally.
> - `LowercaseUrls = true` is the simplest option for lowercase URL generation.
> - These settings affect URL generation, not matching -- matching is always case-insensitive.
