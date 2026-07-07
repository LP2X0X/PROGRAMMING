---
tags:
  - csharp
  - asp-net-core
  - routing
  - constraints
---


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
