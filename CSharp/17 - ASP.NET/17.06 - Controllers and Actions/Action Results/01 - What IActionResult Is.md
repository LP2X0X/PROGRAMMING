---
tags:
  - csharp
  - asp-net-core
  - action-results
  - controllers
---


`IActionResult` is an interface that represents the result of an action method. It defines a single contract:

```csharp
public interface IActionResult
{
    Task ExecuteResultAsync(ActionContext context);
}
```

When the MVC pipeline finishes executing an action method, it takes the returned `IActionResult` and calls `ExecuteResultAsync`. That method is responsible for writing the status code, headers, and body to the `HttpResponse`.

All the convenience methods on `ControllerBase` -- `Ok()`, `NotFound()`, `BadRequest()`, `View()`, and so on -- simply construct and return objects that implement this interface.

```csharp
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetProduct(int id)
    {
        var product = _repository.Find(id);

        if (product is null)
            return NotFound();       // returns NotFoundResult : IActionResult

        return Ok(product);          // returns OkObjectResult : IActionResult
    }
}
```

```ad-note
You never call `ExecuteResultAsync` yourself. The framework does it after your action method returns. Your job is just to return the right `IActionResult` implementation.
```

Under the hood, `ControllerBase.Ok(value)` does something like:

```csharp
public virtual OkObjectResult Ok(object? value)
    => new OkObjectResult(value);
```

And `OkObjectResult.ExecuteResultAsync` writes a 200 status code, serializes the value as JSON (or whatever the configured output formatter produces), and writes it to the response body.
