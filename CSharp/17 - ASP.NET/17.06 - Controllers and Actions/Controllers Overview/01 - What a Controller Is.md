---
tags:
  - csharp
  - asp-net-core
  - controllers
  - mvc
---


A **controller** is a class that handles incoming HTTP requests and produces HTTP responses. It is the **C** in MVC (Model-View-Controller).

Controllers group related request-handling logic into a single class. Each public method on a controller is an **action** -- an endpoint that can be invoked by a client. For example, a `ProductsController` has actions for listing products, fetching a single product, creating, updating, and deleting.

```ad-note
A controller does **not** contain business logic itself. It delegates to services, repositories, and other components. Its job is to:
1. Receive the request
2. Call the appropriate business logic
3. Return the appropriate response
```

The framework discovers controllers automatically during startup by scanning for classes that:
- Inherit from `ControllerBase` or `Controller`
- Have a name ending with the `Controller` suffix
- Are decorated with `[ApiController]` or `[Controller]`

```csharp
// A minimal controller -- each public method is an action
public class ProductsController : ControllerBase
{
    [HttpGet("api/products")]
    public IActionResult GetAll()
    {
        // handle GET /api/products
        return Ok(new[] { "Widget", "Gadget" });
    }

    [HttpGet("api/products/{id}")]
    public IActionResult GetById(int id)
    {
        // handle GET /api/products/5
        return Ok($"Product {id}");
    }
}
```
