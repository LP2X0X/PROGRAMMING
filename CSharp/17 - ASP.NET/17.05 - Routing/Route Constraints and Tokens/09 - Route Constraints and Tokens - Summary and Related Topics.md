---
tags:
  - csharp
  - asp-net-core
  - routing
  - constraints
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
