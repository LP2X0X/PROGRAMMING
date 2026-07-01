---
tags:
  - csharp
  - asp-net-core
  - routing
  - fundamentals
---


When multiple route templates could potentially match an incoming URL, ASP.NET Core uses a set of rules to determine the **best match**.

### Matching Priority

The routing system ranks candidates using **specificity**:

1. **Literal segments** are more specific than parameters
2. **Constrained parameters** are more specific than unconstrained ones
3. **Required parameters** are more specific than optional ones
4. **Catch-all parameters** are the least specific

For example, given these routes:
```
"products/featured"          // All literals
"products/{category}"        // One parameter
"products/{category}/{id:int}" // Two params, one constrained
"{**catchall}"               // Catch-all
```

The URL `/products/featured` matches the first route (literal match wins).
The URL `/products/electronics` matches the second route.
The URL `/products/electronics/42` matches the third route.
The URL `/anything/else/here` matches the catch-all.

### Ambiguous Routes

If two routes have equal specificity for a given URL, ASP.NET Core throws an **`AmbiguousMatchException`** at runtime:

```csharp
app.MapGet("products/{id}", handler1);
app.MapGet("products/{name}", handler2);
// Both match /products/something -- AMBIGUOUS!
```

> [!danger] Critical Warning
> Ambiguous route exceptions happen at **request time**, not at startup. Your application will start successfully, and the error only surfaces when a request triggers the ambiguity. Always test your route configurations with realistic URLs.

### Route Order

You can influence matching order with:
- **The `Order` property** on attribute routes: `[HttpGet("{id}", Order = 1)]`
- **The order of `MapControllerRoute()` calls** -- earlier registrations take priority
- **Convention: more specific routes first**

> [!tip] Practical Tip
> In most applications, you do not need to set `Order` explicitly. The built-in specificity rules handle the vast majority of cases. If you find yourself frequently setting `Order`, it may be a sign that your route structure needs redesigning.

### HTTP Method Matching

For [[Attribute Routing]], the HTTP method is part of the match. `[HttpGet("{id}")]` only matches GET requests. `[HttpPost]` only matches POST. If the URL matches but the method does not, the framework returns **405 Method Not Allowed** (not 404).

For conventional routing, all HTTP methods match by default unless you add method constraints.

> [!summary] Section Summary
> - ASP.NET Core resolves route ambiguity using specificity: literals > constrained params > required params > optional > catch-all.
> - Equally specific routes cause `AmbiguousMatchException` at request time, not startup.
> - Route order can be influenced with `Order` or registration order, but specificity usually suffices.
> - HTTP method matching is part of attribute routing; mismatches produce 405 (not 404).
