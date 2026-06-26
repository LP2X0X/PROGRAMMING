---
title: "Route Constraints and Tokens"
date: 2026-06-18
tags: [csharp, asp-net-core, routing, constraints]
aliases: [Route Constraints, Route Parameter Constraints, Token Replacement, Parameter Transformers]
status: complete
---

# Route Constraints and Tokens

> [!ad-note] Overview
> Route constraints restrict which URLs match a route parameter, acting as a filter at the routing level before the request reaches action code. Token replacement and parameter transformers control how route templates resolve names and format URL segments. This note covers every built-in constraint, custom constraint implementation, token mechanics, and the important distinction between routing constraints and model validation.

---

## Table of Contents

- [What Are Route Constraints](#What%20Are%20Route%20Constraints)
- [Built-In Constraints Reference](#Built-In%20Constraints%20Reference)
- [Combining Constraints](#Combining%20Constraints)
- [Optional Parameters with Constraints](#Optional%20Parameters%20with%20Constraints)
- [Custom Route Constraints](#Custom%20Route%20Constraints)
- [Token Replacement](#Token%20Replacement)
- [Parameter Transformers](#Parameter%20Transformers)
- [Constraints vs Model Validation](#Constraints%20vs%20Model%20Validation)
- [Comprehensive Summary](#Comprehensive%20Summary)
- [Related Topics](#Related%20Topics)

---

## What Are Route Constraints

**Route constraints** are rules applied to route parameters that determine whether a URL segment is a valid match. They act as a **filter during route matching** -- if a constraint fails, the route is skipped and the routing system tries the next candidate route.

### Basic Syntax

Constraints are appended to a parameter with a colon:

```
{parameter:constraint}
```

Example:
```csharp
[HttpGet("products/{id:int}")]
public IActionResult GetById(int id) => Ok();
```

| URL | Matches? | Reason |
|---|---|---|
| `/products/5` | Yes | `5` is a valid integer |
| `/products/42` | Yes | `42` is a valid integer |
| `/products/abc` | No | `abc` is not an integer |
| `/products/3.14` | No | `3.14` is not an integer |

When the constraint fails, the routing system does **not** return a 400 Bad Request. It simply does not match this route and continues evaluating other routes. If no route matches at all, the result is a 404 Not Found.

> [!ad-note] Key Insight
> Constraints operate at the **routing layer**, not the **validation layer**. A constraint failure means "this route does not match this URL." It does not mean "the input is invalid." This distinction is critical for understanding when to use constraints vs. model validation.

> [!summary] Section Summary
> - Route constraints filter URL segments during route matching using the `{param:constraint}` syntax.
> - Failing a constraint skips the route -- it does not produce an error response.
> - Constraints are a routing-level mechanism, not an input validation mechanism.

---

## Built-In Constraints Reference

ASP.NET Core provides a comprehensive set of built-in constraints. Here is the complete reference.

### Type Constraints

These constrain a parameter to a specific data type:

| Constraint | Description | Example | Matches | Does Not Match |
|---|---|---|---|---|
| `int` | 32-bit integer | `{id:int}` | `123`, `-5` | `abc`, `3.14` |
| `long` | 64-bit integer | `{id:long}` | `9223372036854775807` | `abc` |
| `bool` | Boolean | `{active:bool}` | `true`, `false` | `yes`, `1` |
| `decimal` | Decimal number | `{price:decimal}` | `49.99`, `100` | `abc` |
| `double` | Double-precision float | `{lat:double}` | `47.6062`, `-122.33` | `abc` |
| `float` | Single-precision float | `{val:float}` | `3.14` | `abc` |
| `guid` | GUID | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638` | `not-a-guid` |
| `datetime` | DateTime value | `{date:datetime}` | `2026-06-18`, `06/18/2026` | `not-a-date` |

> [!warning] Common Misconception
> The `bool` constraint only matches the literal strings `"true"` and `"false"` (case-insensitive). It does **not** match `"1"`, `"0"`, `"yes"`, or `"no"`. If you need those values, use a custom constraint or handle conversion in the action.

### String Length Constraints

| Constraint | Description | Example | Matches | Does Not Match |
|---|---|---|---|---|
| `minlength(n)` | Minimum string length | `{name:minlength(3)}` | `abc`, `abcdef` | `ab` |
| `maxlength(n)` | Maximum string length | `{name:maxlength(50)}` | `short` | 51+ char string |
| `length(n)` | Exact string length | `{code:length(6)}` | `ABC123` | `ABC` |
| `length(min,max)` | Length range | `{code:length(3,10)}` | `ABC`, `ABCDEFGHIJ` | `AB` |

### Numeric Range Constraints

| Constraint | Description | Example | Matches | Does Not Match |
|---|---|---|---|---|
| `min(n)` | Minimum value | `{age:min(18)}` | `18`, `99` | `17`, `0` |
| `max(n)` | Maximum value | `{quantity:max(999)}` | `1`, `999` | `1000` |
| `range(min,max)` | Value within range | `{age:range(18,120)}` | `18`, `65`, `120` | `17`, `121` |

### Content Constraints

| Constraint | Description | Example | Matches | Does Not Match |
|---|---|---|---|---|
| `alpha` | Alphabetic characters only | `{name:alpha}` | `John`, `abc` | `John123`, `a b` |
| `regex(expr)` | Matches a regex pattern | `{slug:regex(^[a-z0-9-]+$)}` | `my-product` | `My Product!` |
| `required` | Must have a value (not empty) | `{name:required}` | `anything` | (empty string) |

### Special Constraints

| Constraint | Description | Example |
|---|---|---|
| `exists` | Checks if the value matches a known area, controller, etc. | `{area:exists}` |
| `nonfile` | Matches when the value does not look like a filename (no extension) | `{path:nonfile}` |

> [!tip] Practical Tip
> The `regex` constraint is powerful but has a syntax quirk: because route templates use curly braces, you must **double-escape** braces inside regex patterns or avoid them. Also, in attribute routes, square brackets in regex need to be doubled: `[[a-z]]` instead of `[a-z]` because C# attributes use `[` for their own syntax. In `MapControllerRoute()` string patterns, single `[a-z]` is fine.

> [!example] Regex Constraint in Attribute Routing
> ```csharp
> // Note the doubled brackets [[ ]] in attribute routing
> [HttpGet("posts/{slug:regex(^[[a-z0-9-]]+$)}")]
> public IActionResult GetBySlug(string slug) => Ok();
> ```
>
> In conventional route registration (string-based):
> ```csharp
> // Single brackets are fine here
> app.MapControllerRoute(
>     name: "post",
>     pattern: "posts/{slug:regex(^[a-z0-9-]+$)}");
> ```

> [!summary] Section Summary
> - Built-in constraints cover types (`int`, `guid`, `bool`, `datetime`), string lengths, numeric ranges, content (`alpha`, `regex`), and special cases (`exists`, `required`).
> - `bool` only matches `"true"`/`"false"`, not `"1"`/`"0"`.
> - Regex constraints in attribute routes require doubled square brackets `[[ ]]`.
> - The `regex` constraint should be used sparingly; prefer named constraints when possible.

---

## Combining Constraints

Multiple constraints can be applied to a single parameter by chaining them with colons:

```
{parameter:constraint1:constraint2:constraint3}
```

### Examples

```csharp
// id must be an integer AND at least 1
[HttpGet("{id:int:min(1)}")]
public IActionResult GetById(int id) => Ok();

// name must be alphabetic, between 2 and 50 characters
[HttpGet("users/{name:alpha:minlength(2):maxlength(50)}")]
public IActionResult GetByName(string name) => Ok();

// page must be an integer, between 1 and 10000
[HttpGet("list/{page:int:range(1,10000)}")]
public IActionResult ListPage(int page) => Ok();
```

### Evaluation Order

All constraints must pass for the route to match. They are evaluated in the order specified, and evaluation **short-circuits** on the first failure.

> [!tip] Practical Tip
> Put type constraints first (`int`, `guid`, etc.) because they are the cheapest to evaluate and most likely to fail. For example, `{id:int:min(1)}` checks integer first -- if the value is not an integer, `min(1)` is never evaluated.

> [!summary] Section Summary
> - Chain multiple constraints with colons: `{param:int:min(1):max(100)}`.
> - All constraints must pass; evaluation short-circuits on the first failure.
> - Order type constraints first for efficiency.

---

## Optional Parameters with Constraints

Optional parameters (using `?`) can be combined with constraints:

```csharp
// id is optional, but if provided, must be an integer
[HttpGet("products/{id:int?}")]
public IActionResult Get(int? id)
{
    if (id.HasValue)
        return Ok($"Product {id}");
    return Ok("All products");
}
```

| URL | Matches? | id Value |
|---|---|---|
| `/products` | Yes | `null` |
| `/products/5` | Yes | `5` |
| `/products/abc` | No | -- |

### Behavior Details

- When the segment is **absent**, the parameter is `null` (for nullable types) or the default -- the constraint is **not evaluated**
- When the segment is **present**, the constraint **is** evaluated
- The `?` marker goes after all constraints: `{id:int:min(1)?}` -- the parameter is optional, but if present, it must be an integer >= 1

> [!warning] Common Misconception
> `{id:int?}` does not mean "id is an optional integer." It means "id is an optional route segment that, *if present*, must be an integer." The `?` applies to the route segment presence, not to C# nullability. However, you should still use `int?` as the C# parameter type to handle the missing-segment case.

> [!summary] Section Summary
> - Combine `?` with constraints: `{id:int?}` -- optional segment, validated if present.
> - When the segment is absent, the constraint is not evaluated and the value is `null`/default.
> - Place `?` after all constraints in the template.
> - Use a nullable C# type (`int?`) for the corresponding action parameter.

---

## Custom Route Constraints

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

---

## Token Replacement

Token replacement is the mechanism by which special placeholders in route templates are resolved to actual names at application startup.

### The Three Tokens

| Token | Resolves To | Example Class/Method | Result |
|---|---|---|---|
| `[controller]` | Controller class name minus "Controller" suffix | `ProductsController` | `Products` |
| `[action]` | Action method name | `GetById` | `GetById` |
| `[area]` | Area name from `[Area]` attribute | `[Area("Admin")]` | `Admin` |

### How It Works

```csharp
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpGet("[action]/{id}")]
    public IActionResult GetById(int id) => Ok();
}
// Resolved route: GET api/Orders/GetById/{id}
```

### Resolution Timing

Tokens are resolved **once at application startup**, not per-request. The resolved route template is compiled into the route table and used for all subsequent matching. This means:
- Zero runtime overhead for token resolution
- Renaming a class/method changes the route (which is the point)
- You cannot conditionally change token values at runtime

### Token vs Literal Trade-offs

| Approach | Pros | Cons |
|---|---|---|
| `[Route("api/[controller]")]` | Auto-syncs with class name; DRY | Renaming class changes public URL (breaking change) |
| `[Route("api/products")]` | URL is explicit and stable | Manual sync needed; can drift from class name |

> [!tip] Practical Tip
> For internal APIs or early-stage development, `[controller]` tokens are convenient. For **public-facing APIs** where URL stability matters, consider hardcoding the route string. A rename refactoring tool will not know to update your API consumers.

> [!summary] Section Summary
> - `[controller]`, `[action]`, and `[area]` tokens are resolved at startup, not per-request.
> - They keep routes synchronized with code names automatically.
> - For public APIs, hardcoded route strings provide URL stability.
> - Tokens have zero runtime performance cost.

---

## Parameter Transformers

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

---

## Constraints vs Model Validation

This is one of the most frequently confused topics in ASP.NET Core routing. Understanding the distinction is essential.

### Route Constraints: Routing-Level Filtering

Route constraints answer: *"Does this URL match this route?"*

- Run **during route matching**, before the action is selected
- A failing constraint means the route is **skipped** (not that input is invalid)
- The next matching route is tried
- If no route matches, the result is **404 Not Found**

```csharp
[HttpGet("products/{id:int:min(1)}")]
public IActionResult GetById(int id) => Ok();

// /products/abc -> constraint fails -> 404 (no matching route)
// /products/0   -> constraint fails -> 404 (no matching route)
// /products/5   -> constraint passes -> action executes
```

### Model Validation: Business Logic Validation

Model validation answers: *"Is this input acceptable for my business logic?"*

- Runs **after route matching**, inside the action or model binding pipeline
- A validation failure returns **400 Bad Request** with error details
- Uses Data Annotations, `FluentValidation`, or manual checks

```csharp
public class CreateProductDto
{
    [Required]
    [StringLength(200, MinimumLength = 3)]
    public string Name { get; set; }

    [Range(0.01, 999999.99)]
    public decimal Price { get; set; }
}
```

### When to Use Which

| Scenario | Use | Reason |
|---|---|---|
| Parameter must be an integer | Constraint (`{id:int}`) | Prevents non-integer URLs from matching |
| Parameter must be a valid GUID | Constraint (`{id:guid}`) | Prevents non-GUID URLs from matching |
| Price must be > 0 | Validation (`[Range]`) | Business rule, not a routing concern |
| Name must be 3-200 chars | Validation (`[StringLength]`) | Business rule with error feedback |
| ID must be positive | **Either** (depends on context) | Constraint if invalid IDs should 404; validation if they should 400 |

> [!ad-note] Key Insight
> A good rule of thumb: use **route constraints** for values that determine *which resource* is being addressed (type and format of identifiers). Use **model validation** for values that determine *whether the request is acceptable* (business rules, required fields, value ranges on input bodies).

> [!example] Practical Example
> ```csharp
> // Constraint: id must be an integer (routing concern)
> [HttpGet("{id:int}")]
> public IActionResult GetById(int id)
> {
>     // Validation: id must reference an existing product (business concern)
>     var product = _service.GetById(id);
>     if (product is null)
>         return NotFound();
>
>     return Ok(product);
> }
> ```
> Here, `{id:int}` ensures `/products/abc` returns 404 (wrong resource format). But `/products/99999` passes the constraint and reaches the action, where it correctly returns 404 (resource not found).

> [!warning] Common Misconception
> Do **not** use route constraints as a substitute for input validation. Constraint failures produce 404s, not 400s. A user submitting an invalid form field should receive a 400 Bad Request with validation errors, not a confusing 404 Not Found.

> [!summary] Section Summary
> - Route constraints filter at the routing level: failure means "route does not match" (404).
> - Model validation filters at the business level: failure means "input is invalid" (400).
> - Use constraints for identifier format (type, shape). Use validation for business rules.
> - Do not use constraints as a replacement for input validation -- wrong HTTP status and no error details.

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Route constraints** restrict which URL values match a route parameter, acting as filters during the route matching phase. ASP.NET Core provides a rich set of built-in constraints: type constraints (`int`, `guid`, `bool`, `datetime`), string length constraints (`minlength`, `maxlength`, `length`), numeric range constraints (`min`, `max`, `range`), content constraints (`alpha`, `regex`, `required`), and special constraints (`exists`, `nonfile`). Multiple constraints chain with colons (`{id:int:min(1)}`), and optional parameters combine with `?` at the end (`{id:int?}`).
>
> For custom needs, implement `IRouteConstraint`, register it in `ConstraintMap`, and use it like a built-in constraint. Custom constraints must be fast and side-effect-free.
>
> **Token replacement** (`[controller]`, `[action]`, `[area]`) resolves route template placeholders to actual class/method/area names at startup with zero runtime cost. **Parameter transformers** (implementing `IOutboundParameterTransformer`) modify how these tokens appear in generated URLs -- the most common use being PascalCase to kebab-case conversion via a `SlugifyParameterTransformer`. For simple lowercasing, use `options.LowercaseUrls = true`.
>
> The critical distinction to remember: **constraints are for routing** (does this URL match?) while **model validation is for business logic** (is this input acceptable?). Constraint failures produce 404s; validation failures produce 400s. Use constraints for identifier format and type; use validation for business rules and error feedback. See [[Attribute Routing]] for how constraints are used in practice and [[Endpoint Routing]] for how the routing system processes matches.

---

## Related Topics

- [[Routing Overview]]
- [[Attribute Routing]]
- [[Endpoint Routing]]
- [[Model Binding]]
- [[Model Validation]]
- [[Data Annotations]]

---

## Further Reading

- [[Custom Middleware]] -- where you might inspect route values post-matching
- [[Filters in ASP.NET Core]] -- action/result filters that run after routing
- [[Globalization and Localization]] -- URL-based culture selection using constraints
