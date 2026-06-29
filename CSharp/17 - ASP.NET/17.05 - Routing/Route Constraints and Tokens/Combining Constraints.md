---
tags:
  - csharp
  - asp-net-core
  - routing
  - constraints
---


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
