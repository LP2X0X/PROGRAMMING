---
tags:
  - csharp
  - asp-net-core
  - http
  - status-codes
  - web-api
---

## ASP.NET Core and Status Codes

### Action Results and Helper Methods

In ASP.NET Core, controllers return **action results** that encapsulate the status code and response body. The `ControllerBase` class provides helper methods that create these results.

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;

    public OrdersController(IOrderService orderService)
    {
        _orderService = orderService;
    }

    // GET /api/orders → 200 OK
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        var orders = await _orderService.GetAllAsync();
        return Ok(orders);
    }

    // GET /api/orders/5 → 200 OK or 404 Not Found
    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id)
    {
        var order = await _orderService.GetByIdAsync(id);
        return order is null ? NotFound() : Ok(order);
    }

    // POST /api/orders → 201 Created
    [HttpPost]
    public async Task<IActionResult> Create(CreateOrderDto dto)
    {
        var order = await _orderService.CreateAsync(dto);
        return CreatedAtAction(nameof(GetById), new { id = order.Id }, order);
    }

    // PUT /api/orders/5 → 204 No Content or 404 Not Found
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, UpdateOrderDto dto)
    {
        var order = await _orderService.GetByIdAsync(id);
        if (order is null)
            return NotFound();

        await _orderService.UpdateAsync(id, dto);
        return NoContent();
    }

    // DELETE /api/orders/5 → 204 No Content or 404 Not Found
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var existed = await _orderService.DeleteAsync(id);
        return existed ? NoContent() : NotFound();
    }
}
```

> [!ad-note] `IActionResult` vs `ActionResult<T>` vs Concrete Types
> There are three patterns for declaring return types:
> ```csharp
> // 1. IActionResult -- can return any status code, no type info for Swagger
> public IActionResult GetById(int id) => Ok(product);
>
> // 2. ActionResult<T> -- type info for Swagger + can return different status codes
> public ActionResult<Product> GetById(int id)
> {
>     var product = _repo.GetById(id);
>     if (product is null) return NotFound();   // 404
>     return product;                            // 200 (implicit Ok)
> }
>
> // 3. Concrete type -- always 200, no flexibility for error codes
> public Product GetById(int id) => _repo.GetById(id); // Always 200
> ```
> For APIs, `ActionResult<T>` is generally preferred because it gives Swagger the return type while still allowing you to return error codes.

#### Minimal API Equivalents

In [[Minimal APIs]], the helper methods are on the static `Results` (or `TypedResults`) class:

```csharp
var app = WebApplication.Create(args);

app.MapGet("/api/products/{id}", async (int id, IProductRepository repo) =>
{
    var product = await repo.GetByIdAsync(id);
    return product is null
        ? Results.NotFound()                    // 404
        : Results.Ok(product);                  // 200
});

app.MapPost("/api/products", async (CreateProductDto dto, IProductService service) =>
{
    var product = await service.CreateAsync(dto);
    return Results.Created($"/api/products/{product.Id}", product); // 201
});

app.MapDelete("/api/products/{id}", async (int id, IProductRepository repo) =>
{
    var existed = await repo.DeleteAsync(id);
    return existed
        ? Results.NoContent()                   // 204
        : Results.NotFound();                   // 404
});
```

---

### Problem Details for Error Responses

[[Problem Details]] (RFC 9457, formerly RFC 7807) is the standard format for returning machine-readable error information in HTTP APIs. ASP.NET Core has built-in support.

```json
{
    "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
    "title": "Bad Request",
    "status": 400,
    "detail": "The 'Price' field must be greater than 0.",
    "instance": "/api/products",
    "traceId": "00-abc123-def456-01"
}
```

Enable Problem Details globally:

```csharp
// In Program.cs
builder.Services.AddProblemDetails();

// The [ApiController] attribute automatically uses Problem Details
// for 400 validation errors and other client errors
```

> [!ad-tip] Consistent Error Responses
> With Problem Details enabled, all your 4xx and 5xx responses follow a consistent JSON structure. This makes it much easier for API consumers to handle errors programmatically -- they always know where to find the error message, status code, and trace ID.

---

### Exception Handling Middleware

The [[Exception Handling]] middleware catches unhandled exceptions and converts them to 500 responses. Without it, an unhandled exception would crash the request with no response.

```csharp
var app = builder.Build();

// Add exception handling FIRST in the pipeline
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage(); // Detailed error page (HTML)
}
else
{
    app.UseExceptionHandler(); // Returns Problem Details JSON for APIs
}
```

In development, `UseDeveloperExceptionPage()` shows a detailed HTML error page with the stack trace. In production, `UseExceptionHandler()` returns a sanitized error response (no stack traces leaked to clients).

---

### Middleware Short-Circuiting

Any [[Middleware Overview|middleware]] in the pipeline can ==short-circuit the request== and return a status code directly, without reaching your controller.

```csharp
// Custom middleware that rejects requests without an API key
app.Use(async (context, next) =>
{
    if (!context.Request.Headers.ContainsKey("X-Api-Key"))
    {
        context.Response.StatusCode = 401; // Short-circuit with 401
        await context.Response.WriteAsJsonAsync(new
        {
            error = "API key is required. Include an X-Api-Key header."
        });
        return; // Do NOT call next() -- request stops here
    }

    await next(context); // API key present, continue to next middleware
});
```

Common middleware that returns status codes:

| Middleware | Status Code | When |
|---|---|---|
| Authentication | 401 | No valid credentials |
| Authorization | 403 | Insufficient permissions |
| Rate Limiting | 429 | Too many requests |
| CORS | 403/204 | Origin not allowed / preflight |
| Static Files | 200/304/404 | File found/cached/missing |
| HTTPS Redirection | 307 | HTTP request redirected to HTTPS |

> [!summary] Section Summary
> - `ControllerBase` provides helper methods like `Ok()`, `NotFound()`, `BadRequest()` that set status codes
> - `ActionResult<T>` is preferred over `IActionResult` for APIs because it provides type information for Swagger
> - Minimal APIs use `Results.Ok()`, `Results.NotFound()`, etc. instead of controller helpers
> - [[Problem Details]] standardizes error response format for 4xx/5xx codes
> - [[Exception Handling]] middleware catches unhandled exceptions and returns 500
> - Any middleware can short-circuit the pipeline and return a status code directly
