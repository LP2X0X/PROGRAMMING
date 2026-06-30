---
tags:
  - csharp
  - asp-net-core
  - routing
  - attribute-routing
---


## Comprehensive Summary

> [!tip] Complete Summary
> **Attribute routing** is the mechanism of declaring URL routes directly on controller classes and action methods using attributes like `[Route]`, `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]`, and `[HttpPatch]`. It is **required** for controllers decorated with `[ApiController]` and is the standard approach for Web APIs.
>
> Routes follow a **composition model**: the controller-level `[Route]` sets a prefix, and action-level templates are appended. Action templates beginning with `/` override the controller prefix. **Token replacement** (`[controller]`, `[action]`, `[area]`) dynamically inserts names resolved at startup, keeping routes synchronized with code.
>
> Multiple route attributes can be stacked on a single action or controller, producing a Cartesian product of all combinations. **Route names** (e.g., `Name = "GetProduct"`) enable URL generation via `Url.Action()`, `Url.RouteUrl()`, `CreatedAtRoute()`, and `LinkGenerator`. Names must be globally unique.
>
> The `Order` property controls evaluation priority but is rarely needed thanks to the built-in specificity algorithm. **Areas** add organizational grouping, supported via `[Area]` and the `[area]` token. Compared to [[Routing Overview|conventional routing]], attribute routing is more explicit and self-documenting, making it the preferred choice for most modern ASP.NET Core applications. For route parameter validation at the routing level (before the action runs), see [[Route Constraints and Tokens]].

---

## Related Topics

- [[Routing Overview]]
- [[Endpoint Routing]]
- [[Route Constraints and Tokens]]
- [[Model Binding]]
- [[Web API Design]]
- [[Controllers and Actions]]
- [[URL Generation]]

---

## Further Reading

- [[Minimal APIs]] -- an alternative to controllers that also uses attribute-style routing
- [[API Versioning]] -- how to version attribute-routed APIs
- [[Content Negotiation]] -- how `[Produces]` and `[Consumes]` work with routing
- [[Authentication and Authorization]] -- how `[Authorize]` interacts with routed endpoints
