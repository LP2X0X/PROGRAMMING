---
tags:
  - csharp
  - asp-net-core
  - routing
  - constraints
---


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
