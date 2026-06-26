---
tags:
 - csharp
 - asp-net-core
 - action-results
 - controllers
---

# Action Results

Action results are how controller actions tell the MVC pipeline what HTTP response to send back to the client. Every helper method like `Ok()`, `NotFound()`, `View()` returns an object that implements `IActionResult`, which the framework then executes to produce the actual HTTP response.

---

## What IActionResult Is

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

---

## ActionResult\<T\> -- The Generic Version

`ActionResult<T>` combines the flexibility of `IActionResult` (returning different status codes) with strong typing on the success path.

### Why It Exists

With plain `IActionResult`, the framework has no idea what type the success response contains. This matters for:

- **Swagger/OpenAPI generation** -- tools like Swashbuckle need to know the response schema
- **Compile-time safety** -- you get no type checking on what you pass to `Ok()`

`ActionResult<T>` solves both problems.

### How It Works

`ActionResult<T>` supports implicit conversion from two sources:

1. **From `T` directly** -- auto-wrapped in an `OkObjectResult` (200)
2. **From any `ActionResult`** -- used as-is (e.g., `NotFound()`, `BadRequest()`)

```csharp
public class ProductsController : ControllerBase
{
    private readonly IProductRepository _repository;

    public ProductsController(IProductRepository repository)
    {
        _repository = repository;
    }

    [HttpGet("{id}")]
    public ActionResult<Product> GetProduct(int id)
    {
        var product = _repository.Find(id);

        if (product is null)
            return NotFound();   // implicit conversion from ActionResult

        return product;          // implicit conversion from T (auto-wrapped in Ok)
    }
}
```

```ad-tip
`ActionResult<T>` is the recommended return type for API actions. It gives you the best of both worlds: multiple status codes and strong typing for the success path.
```

### OpenAPI Benefits

When you use `ActionResult<Product>`, the framework automatically produces `[ProducesResponseType(typeof(Product), 200)]` metadata. Swagger picks this up without you needing to annotate every action:

```csharp
// With IActionResult, Swagger doesn't know the success type:
[HttpGet("{id}")]
[ProducesResponseType(typeof(Product), 200)]   // you must add this manually
[ProducesResponseType(404)]
public IActionResult GetProduct(int id) { ... }

// With ActionResult<T>, Swagger infers the 200 response type:
[HttpGet("{id}")]
[ProducesResponseType(404)]                     // only need to declare non-success codes
public ActionResult<Product> GetProduct(int id) { ... }
```

---

## Complete Table of Common Result Types

| Helper Method | Status Code | Return Type | When to Use |
|---|---|---|---|
| `Ok()` | 200 | `OkResult` | Successful action, no body |
| `Ok(value)` | 200 | `OkObjectResult` | Successful GET, return data |
| `Created(uri, value)` | 201 | `CreatedResult` | Resource created, provide location URI |
| `CreatedAtAction(action, routeValues, value)` | 201 | `CreatedAtActionResult` | Resource created, location header points to a GET action |
| `CreatedAtRoute(routeName, routeValues, value)` | 201 | `CreatedAtRouteResult` | Resource created, location header uses a named route |
| `Accepted()` | 202 | `AcceptedResult` | Long-running operation accepted for processing |
| `NoContent()` | 204 | `NoContentResult` | Successful PUT/DELETE with no response body |
| `BadRequest()` | 400 | `BadRequestResult` | Validation failure, no details |
| `BadRequest(modelState)` | 400 | `BadRequestObjectResult` | Validation failure with error details |
| `Unauthorized()` | 401 | `UnauthorizedResult` | Client is not authenticated |
| `Forbid()` | 403 | `ForbidResult` | Authenticated but lacks permission |
| `NotFound()` | 404 | `NotFoundResult` | Resource does not exist, no details |
| `NotFound(value)` | 404 | `NotFoundObjectResult` | Resource does not exist, with error info |
| `Conflict()` | 409 | `ConflictResult` | Concurrency or state conflict, no details |
| `Conflict(value)` | 409 | `ConflictObjectResult` | Concurrency conflict with error info |
| `UnprocessableEntity()` | 422 | `UnprocessableEntityObjectResult` | Semantic validation failure |
| `StatusCode(code)` | any | `StatusCodeResult` | Custom or uncommon status codes |
| `StatusCode(code, value)` | any | `ObjectResult` | Custom status code with body |
| `View()` / `View(model)` | 200 | `ViewResult` | Render a Razor view (MVC only) |
| `PartialView()` | 200 | `PartialViewResult` | Render a partial Razor view |
| `RedirectToAction(action)` | 302 | `RedirectToActionResult` | Redirect to another action method |
| `RedirectToPage(pageName)` | 302 | `RedirectToPageResult` | Redirect to a Razor Page |
| `Redirect(url)` | 302 | `RedirectResult` | Redirect to an external or absolute URL |
| `RedirectPermanent(url)` | 301 | `RedirectResult` | Permanent redirect (SEO) |
| `Json(value)` | 200 | `JsonResult` | Force JSON serialization explicitly |
| `File(bytes, contentType)` | 200 | `FileContentResult` | File download from byte array |
| `File(stream, contentType)` | 200 | `FileStreamResult` | File download from stream |
| `PhysicalFile(path, contentType)` | 200 | `PhysicalFileResult` | File download from disk path |
| `Content(text)` | 200 | `ContentResult` | Plain text response |

```ad-info
`Forbid()` and `Unauthorized()` differ in an important way. `Unauthorized()` returns a raw 401 and does nothing else. `Forbid()` triggers the configured authentication handler's `ForbidAsync` method, which can redirect to an access-denied page. For API controllers, `Forbid()` also produces a 403, but through the authentication middleware.
```

```ad-warning
`Json()` is rarely needed when using `[ApiController]`. The framework already serializes objects as JSON via content negotiation. Use `Json()` only when you need to override serializer settings for a specific response or when returning JSON from an MVC controller that normally returns views.
```

---

## When to Use IActionResult vs ActionResult\<T\> vs Concrete Type

There are three ways to declare an action method's return type. Each has its place:

### Return T Directly

Use when the action always succeeds with 200. Rare in practice because most endpoints need error handling.

```csharp
[HttpGet]
public IEnumerable<Product> GetAllProducts()
{
    return _repository.GetAll();
}
```

If the method throws, the exception middleware handles it. There is no way to return a 404 or 400 from this signature without throwing.

### Return IActionResult

Use when you need multiple status codes and don't care about strong typing on the success path.

```csharp
[HttpGet("{id}")]
public IActionResult GetProduct(int id)
{
    var product = _repository.Find(id);

    if (product is null)
        return NotFound();

    return Ok(product);
}
```

Works, but Swagger cannot infer the success response type without `[ProducesResponseType]` attributes.

### Return ActionResult\<T\> (Recommended for APIs)

Use when you want both multiple status codes and strong typing.

```csharp
[HttpGet("{id}")]
public ActionResult<Product> GetProduct(int id)
{
    var product = _repository.Find(id);

    if (product is null)
        return NotFound();

    return product;    // implicit conversion, auto-wrapped in Ok
}
```

```ad-tip
**Rule of thumb for APIs:**
- Default to `ActionResult<T>` for most endpoints.
- Use `IActionResult` when the success type genuinely varies (e.g., file downloads that might also return JSON errors).
- Use `T` directly only for trivial endpoints that never fail.
```

### Async Actions

All three approaches work with `Task<>`:

```csharp
// Concrete type
public async Task<Product> GetProduct(int id) { ... }

// IActionResult
public async Task<IActionResult> GetProduct(int id) { ... }

// ActionResult<T> (recommended)
public async Task<ActionResult<Product>> GetProduct(int id) { ... }
```

---

## Implicit Conversion in ActionResult\<T\>

The ergonomics of `ActionResult<T>` come from two implicit operators defined on the struct:

```csharp
// Simplified -- what the framework defines:
public sealed class ActionResult<TValue>
{
    // Implicit from TValue -- wraps in OkObjectResult
    public static implicit operator ActionResult<TValue>(TValue value);

    // Implicit from ActionResult -- discards T, uses the action result as-is
    public static implicit operator ActionResult<TValue>(ActionResult result);
}
```

This means you can write natural, clean code:

```csharp
[HttpGet("{id}")]
public ActionResult<Product> GetProduct(int id)
{
    var product = _repository.Find(id);

    if (product is null)
        return NotFound();       // ActionResult -> ActionResult<Product>

    return product;              // Product -> ActionResult<Product> (wrapped in Ok)
}
```

### What Happens at Runtime

When you `return product;`:
1. The implicit operator wraps `product` in a new `ActionResult<Product>` that stores the value.
2. The framework detects the value, creates an `OkObjectResult`, and executes it.
3. The response is 200 with the product serialized as JSON.

When you `return NotFound();`:
1. `NotFound()` returns a `NotFoundResult` (which is an `ActionResult`).
2. The implicit operator wraps it in `ActionResult<Product>`, but the `Product` type is irrelevant.
3. The framework detects the `ActionResult`, ignores `T`, and executes the `NotFoundResult`.
4. The response is 404 with no body.

```ad-note
You cannot `return null;` from `ActionResult<T>` when `T` is a reference type. It compiles, but at runtime it produces a 204 No Content response (the framework treats a null value as "nothing to serialize"). If you want a 404, explicitly return `NotFound()`.
```

---

## ProblemDetails

ProblemDetails is a standardized format (RFC 7807 / RFC 9457) for returning machine-readable error information in HTTP API responses. ASP.NET Core has built-in support for it.

### What a ProblemDetails Response Looks Like

```json
{
    "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
    "title": "Not Found",
    "status": 404,
    "detail": "Product with ID 42 was not found.",
    "traceId": "00-abc123def456-789ghi-00"
}
```

For validation errors, the framework uses `ValidationProblemDetails`, which adds an `errors` dictionary:

```json
{
    "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "errors": {
        "Name": ["The Name field is required."],
        "Price": ["The Price must be greater than 0."]
    },
    "traceId": "00-abc123def456-789ghi-00"
}
```

### Automatic ProblemDetails with \[ApiController\]

When you apply `[ApiController]` to a controller, the framework automatically:

1. Converts `ModelState` validation failures into `ValidationProblemDetails` (400)
2. Wraps client error status codes (4xx) in `ProblemDetails` format

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    public ActionResult<Product> GetProduct(int id)
    {
        var product = _repository.Find(id);

        if (product is null)
            return NotFound();   // automatically formatted as ProblemDetails JSON

        return product;
    }
}
```

### Enabling ProblemDetails Globally

In `Program.cs`, you can configure ProblemDetails for the entire application, including for exceptions:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Enable ProblemDetails for all error responses, including unhandled exceptions
builder.Services.AddProblemDetails();

var app = builder.Build();

// The exception handler and status code pages use ProblemDetails format
app.UseExceptionHandler();
app.UseStatusCodePages();

app.MapControllers();
app.Run();
```

### Customizing ProblemDetails

You can customize the ProblemDetails output by configuring `ProblemDetailsOptions`:

```csharp
builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = context =>
    {
        context.ProblemDetails.Instance =
            $"{context.HttpContext.Request.Method} {context.HttpContext.Request.Path}";

        context.ProblemDetails.Extensions["requestId"] =
            context.HttpContext.TraceIdentifier;

        context.ProblemDetails.Extensions["serverTime"] =
            DateTime.UtcNow.ToString("O");
    };
});
```

### Returning ProblemDetails Manually

You can also return `ProblemDetails` explicitly using the `Problem()` helper:

```csharp
[HttpGet("{id}")]
public ActionResult<Product> GetProduct(int id)
{
    var product = _repository.Find(id);

    if (product is null)
    {
        return Problem(
            detail: $"Product with ID {id} was not found.",
            statusCode: StatusCodes.Status404NotFound,
            title: "Product Not Found"
        );
    }

    return product;
}
```

```ad-attention
By default, in `Development` environment, ProblemDetails may include the exception stack trace for 500 errors. In production, it omits sensitive details. Make sure you test both environments to verify what clients actually see.
```

---

## Real-World Example -- OrdersController

Below is a complete controller demonstrating all the common action result patterns for a typical CRUD API. This uses `ActionResult<T>` throughout.

```csharp
using Microsoft.AspNetCore.Mvc;

namespace MyStore.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderRepository _orderRepository;
    private readonly IProductRepository _productRepository;

    public OrdersController(
        IOrderRepository orderRepository,
        IProductRepository productRepository)
    {
        _orderRepository = orderRepository;
        _productRepository = productRepository;
    }

    // -------------------------------------------------------
    // GET api/orders
    // Returns: 200 Ok with list of orders
    // -------------------------------------------------------
    [HttpGet]
    public async Task<ActionResult<IEnumerable<OrderSummaryDto>>> GetOrders(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 20)
    {
        var orders = await _orderRepository.GetPagedAsync(page, pageSize);
        return Ok(orders);
    }

    // -------------------------------------------------------
    // GET api/orders/{id}
    // Returns: 200 Ok with order, or 404 Not Found
    // -------------------------------------------------------
    [HttpGet("{id}")]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<OrderDetailDto>> GetOrder(int id)
    {
        var order = await _orderRepository.GetByIdAsync(id);

        if (order is null)
            return NotFound();

        return order;
    }

    // -------------------------------------------------------
    // POST api/orders
    // Returns: 201 Created with location header, or 400 Bad Request
    // -------------------------------------------------------
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<OrderDetailDto>> CreateOrder(
        CreateOrderRequest request)
    {
        // [ApiController] handles model state validation automatically.
        // If we reach here, model state is valid.

        // Business validation
        foreach (var item in request.Items)
        {
            var product = await _productRepository.GetByIdAsync(item.ProductId);
            if (product is null)
            {
                ModelState.AddModelError(
                    nameof(item.ProductId),
                    $"Product {item.ProductId} does not exist.");

                return BadRequest(ModelState);
            }
        }

        var order = await _orderRepository.CreateAsync(request);

        // Returns 201 with a Location header like:
        //   Location: https://mystore.com/api/orders/42
        return CreatedAtAction(
            actionName: nameof(GetOrder),
            routeValues: new { id = order.Id },
            value: order);
    }

    // -------------------------------------------------------
    // PUT api/orders/{id}
    // Returns: 204 No Content, 404 Not Found, or 409 Conflict
    // -------------------------------------------------------
    [HttpPut("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status409Conflict)]
    public async Task<IActionResult> UpdateOrder(
        int id,
        UpdateOrderRequest request)
    {
        var existingOrder = await _orderRepository.GetByIdAsync(id);

        if (existingOrder is null)
            return NotFound();

        // Concurrency check using a version/timestamp
        if (existingOrder.Version != request.ExpectedVersion)
        {
            return Conflict(new
            {
                Message = "The order was modified by another user.",
                CurrentVersion = existingOrder.Version
            });
        }

        await _orderRepository.UpdateAsync(id, request);

        return NoContent();
    }

    // -------------------------------------------------------
    // DELETE api/orders/{id}
    // Returns: 204 No Content, or 404 Not Found
    // -------------------------------------------------------
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DeleteOrder(int id)
    {
        var order = await _orderRepository.GetByIdAsync(id);

        if (order is null)
            return NotFound();

        await _orderRepository.DeleteAsync(id);

        return NoContent();
    }

    // -------------------------------------------------------
    // POST api/orders/{id}/cancel
    // Returns: 200 Ok, 404 Not Found, or 422 Unprocessable Entity
    // -------------------------------------------------------
    [HttpPost("{id}/cancel")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status422UnprocessableEntity)]
    public async Task<ActionResult<OrderDetailDto>> CancelOrder(int id)
    {
        var order = await _orderRepository.GetByIdAsync(id);

        if (order is null)
            return NotFound();

        if (order.Status == OrderStatus.Shipped)
        {
            return UnprocessableEntity(new
            {
                Message = "Cannot cancel an order that has already shipped."
            });
        }

        var cancelledOrder = await _orderRepository.CancelAsync(id);

        return Ok(cancelledOrder);
    }

    // -------------------------------------------------------
    // GET api/orders/{id}/invoice
    // Returns: 200 Ok (PDF file), or 404 Not Found
    // -------------------------------------------------------
    [HttpGet("{id}/invoice")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DownloadInvoice(int id)
    {
        var order = await _orderRepository.GetByIdAsync(id);

        if (order is null)
            return NotFound();

        byte[] pdfBytes = await _orderRepository.GenerateInvoicePdfAsync(id);

        return File(
            pdfBytes,
            "application/pdf",
            $"invoice-{id}.pdf");
    }

    // -------------------------------------------------------
    // POST api/orders/{id}/process
    // Returns: 202 Accepted (long-running operation)
    // -------------------------------------------------------
    [HttpPost("{id}/process")]
    [ProducesResponseType(StatusCodes.Status202Accepted)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> ProcessOrder(int id)
    {
        var order = await _orderRepository.GetByIdAsync(id);

        if (order is null)
            return NotFound();

        // Queue for background processing instead of doing it synchronously
        await _orderRepository.QueueForProcessingAsync(id);

        return Accepted(
            uri: Url.Action(nameof(GetOrder), new { id }),
            value: new { Message = "Order processing has been queued.", OrderId = id });
    }
}
```

```ad-summary
**Pattern recap for the OrdersController above:**
- **GET collection** -- `Ok(list)` always (200)
- **GET single** -- `Ok(item)` or `NotFound()` (200 / 404)
- **POST create** -- `CreatedAtAction(...)` or `BadRequest(modelState)` (201 / 400)
- **PUT update** -- `NoContent()`, `NotFound()`, or `Conflict(...)` (204 / 404 / 409)
- **DELETE** -- `NoContent()` or `NotFound()` (204 / 404)
- **Custom action** -- `Ok(result)`, `NotFound()`, or `UnprocessableEntity(...)` (200 / 404 / 422)
- **File download** -- `File(bytes, contentType, fileName)` or `NotFound()` (200 / 404)
- **Long-running** -- `Accepted(uri, value)` or `NotFound()` (202 / 404)

Note that PUT and DELETE return `IActionResult` instead of `ActionResult<T>` because their success path (204 No Content) has no body to type.
```

---

## See Also

- [[Controllers Overview]] -- controller basics, routing, `[ApiController]` attribute
- [[Model Binding]] -- how request data maps to action parameters
- [[Validation]] -- model validation and `ModelState`
- [[Filters]] -- action filters, result filters, and how they wrap action results
- [[17.07 - Views and Razor]] -- `ViewResult`, `PartialViewResult`, and Razor rendering
- [[17.08 - Web APIs]] -- API-specific patterns, content negotiation, and formatters
