---
tags:
  - csharp
  - asp-net-core
  - routing
  - constraints
---


**Route constraints** are rules applied to [[Route Parameters|route parameters]] that determine whether a URL segment is a valid match. They act as a **filter during route matching** -- if a constraint fails, the route is skipped and the routing system tries the next candidate route.

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
