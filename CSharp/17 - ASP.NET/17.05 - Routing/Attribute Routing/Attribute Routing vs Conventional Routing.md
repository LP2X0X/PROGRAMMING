---
tags:
  - csharp
  - asp-net-core
  - routing
  - attribute-routing
---


Both approaches are valid, and understanding when to use each is important.

| Aspect | Attribute Routing | Conventional Routing |
|---|---|---|
| Route location | On controllers/actions | Centralized in `Program.cs` |
| Explicitness | Fully explicit | Convention-based |
| URL flexibility | Complete control | Limited by `{controller}/{action}` pattern |
| API suitability | Excellent (required for `[ApiController]`) | Poor for APIs |
| MVC View suitability | Good | Good (often simpler) |
| Discoverability | Look at the controller | Look at the route table |
| Refactoring impact | Route stays with the code | Centralized routes may break if controllers rename |
| `[ApiController]` compatible | Yes (required) | No |

### Guidelines

- **Web APIs**: Always use attribute routing. The `[ApiController]` attribute requires it, and explicit routes are essential for RESTful design.
- **MVC with Views**: Either works. Conventional routing is often simpler when your URLs follow the standard `{controller}/{action}/{id?}` pattern. Attribute routing when you need custom URL structures.
- **Mixed Applications**: Use both. API controllers use attribute routing; MVC controllers use conventional routing or attribute routing as needed.

> [!ad-note] Key Insight
> In practice, most modern ASP.NET Core applications use attribute routing exclusively. Conventional routing is primarily useful for traditional MVC applications with predictable URL structures. New projects tend to favor attribute routing for its explicitness and flexibility.

> [!summary] Section Summary
> - Attribute routing is explicit and required for `[ApiController]`; conventional routing is convention-based and centralized.
> - Web APIs should always use attribute routing; MVC views can use either.
> - Most modern applications favor attribute routing for its self-documenting nature.
> - Both systems can coexist in the same application.
