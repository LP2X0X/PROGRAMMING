---
tags:
  - csharp
  - asp-net-core
  - routing
  - fundamentals
---


## Comprehensive Summary

> [!tip] Complete Summary
> **Routing** in ASP.NET Core is the system that matches incoming HTTP requests to endpoint handlers. It evolved from two separate systems -- **conventional routing** (centralized templates with `{controller}/{action}` conventions) and **[[Attribute Routing]]** (route attributes on controllers/actions) -- into a unified **[[Endpoint Routing]]** system introduced in .NET Core 3.0.
>
> Endpoint routing uses a **two-phase model**: `UseRouting()` selects the best matching endpoint, and the terminal middleware (`MapControllers()`, `MapRazorPages()`, `MapGet()`, etc.) executes it. Middleware between these two phases can inspect the selected endpoint's metadata, enabling authorization, CORS, and rate limiting to work uniformly across all endpoint types.
>
> **Route templates** define URL patterns using literal segments, parameters (`{id}`), optional parameters (`{id?}`), defaults (`{action=Index}`), catch-all parameters (`{**slug}`), and **[[Route Constraints and Tokens|constraints]]** (`{id:int}`). When multiple routes match, ASP.NET Core resolves the conflict using specificity rules where literals beat parameters and constrained parameters beat unconstrained ones.
>
> The middleware pipeline has three zones relative to routing: pre-routing (no endpoint info), between routing and execution (endpoint selected but not run), and execution (the endpoint runs). In .NET 6+, `UseRouting()` is implicit but should be called explicitly when precise middleware ordering matters.

---

## Related Topics

- [[Attribute Routing]]
- [[Endpoint Routing]]
- [[Route Constraints and Tokens]]
- [[Middleware Pipeline]]
- [[Minimal APIs]]
- [[Model Binding]]
- [[URL Generation]]
- [[Areas in MVC]]

---

## Further Reading

- [[Controllers and Actions]]
- [[Razor Pages]]
- [[Web API Design]]
- [[Authentication and Authorization]]
- [[CORS in ASP.NET Core]]
