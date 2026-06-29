---
tags:
  - csharp
  - asp-net-core
  - routing
  - constraints
---


When built-in constraints are insufficient, you can implement `IRouteConstraint` to create your own.

### Step 1: Implement `IRouteConstraint`

```csharp
using System.Text.RegularExpressions;

public class EvenNumberConstraint : IRouteConstraint
{
    public bool Match(
        HttpContext? httpContext,
        IRouter? route,
        string routeKey,
        RouteValueDictionary values,
        RouteDirection routeDirection)
    {
        if (!values.TryGetValue(routeKey, out var value))
            return false;

        if (!int.TryParse(value?.ToString(), out var number))
            return false;

        return number % 2 == 0;
    }
}
```

### Step 2: Register the Constraint

```csharp
builder.Services.AddRouting(options =>
{
    options.ConstraintMap.Add("even", typeof(EvenNumberConstraint));
});
```

### Step 3: Use It in Route Templates

```csharp
[HttpGet("items/{id:even}")]
public IActionResult GetEvenItem(int id) => Ok();
// Matches: /items/2, /items/42
// Does not match: /items/3, /items/abc
```

### Real-World Custom Constraint Example

A constraint that only matches valid ISO 639-1 language codes:

```csharp
public class LanguageCodeConstraint : IRouteConstraint
{
    private static readonly HashSet<string> ValidCodes = new(StringComparer.OrdinalIgnoreCase)
    {
        "en", "fr", "de", "es", "ja", "zh", "ko", "pt", "ru", "ar"
        // ... add all supported languages
    };

    public bool Match(
        HttpContext? httpContext,
        IRouter? route,
        string routeKey,
        RouteValueDictionary values,
        RouteDirection routeDirection)
    {
        if (!values.TryGetValue(routeKey, out var value))
            return false;

        return ValidCodes.Contains(value?.ToString() ?? string.Empty);
    }
}
```

```csharp
// Registration
builder.Services.AddRouting(options =>
{
    options.ConstraintMap.Add("lang", typeof(LanguageCodeConstraint));
});

// Usage
[HttpGet("{lang:lang}/products")]
public IActionResult GetLocalizedProducts(string lang) => Ok();
// Matches: /en/products, /fr/products
// Does not match: /xx/products, /english/products
```

> [!tip] Practical Tip
> Custom constraints should be **fast and side-effect-free**. They run during route matching, which happens for every request. Do not hit a database or call an external service inside a constraint. If you need that level of validation, do it in the action or a filter.

> [!summary] Section Summary
> - Implement `IRouteConstraint` for custom matching logic.
> - Register with `options.ConstraintMap.Add("name", typeof(...))`.
> - Use in templates as `{param:name}`.
> - Custom constraints must be fast and side-effect-free -- no I/O, no database calls.
