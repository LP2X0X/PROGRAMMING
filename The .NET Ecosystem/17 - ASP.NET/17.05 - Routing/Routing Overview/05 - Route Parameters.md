---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---

A route parameter is a placeholder in the URL path that captures a value from the request URL. It works the same way as in controller-based routing, but the binding happens via the handler's method parameters.

![[Pasted image 20260630085302.png|center]]

### Basic Route Parameters

```csharp
// Simple parameter
app.MapGet("/products/{id}", (int id) =>
    Results.Ok($"Product ID: {id}"));

// Multiple parameters
app.MapGet("/categories/{categoryId}/products/{productId}",
    (int categoryId, int productId) =>
        Results.Ok($"Category: {categoryId}, Product: {productId}"));

// String parameter (no conversion needed)
app.MapGet("/products/search/{name}", (string name) =>
    Results.Ok($"Searching for: {name}"));
```

### Route Constraints

You can apply **route constraints** to restrict what values match a parameter:

```csharp
// Only matches if id is an integer
app.MapGet("/products/{id:int}", (int id) =>
    Results.Ok($"Product {id}"));

// Only matches if id is a GUID
app.MapGet("/orders/{id:guid}", (Guid id) =>
    Results.Ok($"Order {id}"));

// Minimum value constraint
app.MapGet("/products/{id:int:min(1)}", (int id) =>
    Results.Ok($"Product {id}"));

// Regex constraint
app.MapGet("/products/{sku:regex(^[A-Z]{{2}}-\\d{{4}}$)}", (string sku) =>
    Results.Ok($"SKU: {sku}"));

// Length constraint
app.MapGet("/products/{code:length(5)}", (string code) =>
    Results.Ok($"Code: {code}"));
```

### Common Route Constraints

| Constraint     | Example                    | Matches                       |
|---|---|---|
| `int`          | `{id:int}`                 | `123`, `-1`                   |
| `bool`         | `{active:bool}`            | `true`, `false`               |
| `datetime`     | `{date:datetime}`          | `2026-06-18`                  |
| `decimal`      | `{price:decimal}`          | `49.99`                       |
| `guid`         | `{id:guid}`                | `CD2C1638-1638-72D5-...`      |
| `long`         | `{id:long}`                | `9223372036854775807`         |
| `minlength(n)` | `{name:minlength(3)}`      | Any string with 3+ chars      |
| `maxlength(n)` | `{name:maxlength(50)}`     | Any string with 50 or fewer   |
| `range(m,n)`   | `{age:range(18,120)}`      | Integer between 18 and 120    |
| `alpha`        | `{name:alpha}`             | Alphabetic characters only    |
| `required`     | `{name:required}`          | Non-empty value               |

### Catch-All Parameters

These parameters match the remaining unmatched portion of a URL, including any slashes or other characters that aren’t part of earlier parameters.

```csharp
// Catch-all: matches any remaining path segments
app.MapGet("/files/{*filePath}", (string filePath) =>
    Results.Ok($"File path: {filePath}"));
// GET /files/images/photo.jpg -> filePath = "images/photo.jpg"
```

```ad-note
The one- and two-asterisk versions of the catch-all parameter behave identically when routing an incoming request to an endpoint. The difference occurs only when you’re generating URLs: the one-asterisk version URL encodes forward slashes, and the two-asterisk version doesn’t. Typically, the round-trip behavior of the two-asterisk version is what you want.
```

### Optional Parameters

```csharp
// Optional route parameter with nullable type
app.MapGet("/products/{id:int?}", (int? id) =>
    id.HasValue
        ? Results.Ok($"Product {id}")
        : Results.Ok("All products"));
```

> [!summary] Section Summary
> Route parameters are declared in the route pattern with `{name}` syntax and bound to handler parameters by name. Constraints like `:int`, `:guid`, `:min(1)` restrict matches. Catch-all parameters use `{*name}`, and optional parameters use `?` with nullable types.
