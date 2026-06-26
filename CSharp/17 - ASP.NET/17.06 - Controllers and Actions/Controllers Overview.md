---
tags:
 - csharp
 - asp-net-core
 - controllers
 - mvc
---

## What a Controller Is

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

---

## Controller vs ControllerBase

ASP.NET Core provides two base classes for controllers. Choosing the right one depends on whether you are building an **API** or an **MVC app with views**.

### Class Hierarchy

```
Microsoft.AspNetCore.Mvc.ControllerBase
    |
    +-- Microsoft.AspNetCore.Mvc.Controller
```

`Controller` inherits from `ControllerBase` and adds view-rendering capabilities on top.

### ControllerBase (for APIs)

`ControllerBase` is the lean base class designed for API controllers. It provides everything you need to handle HTTP requests and produce responses without any view-rendering overhead.

**Key Properties:**

| Property      | Description                                               |
| ------------- | --------------------------------------------------------- |
| `HttpContext` | The full HTTP context for the current request             |
| `Request`     | The `HttpRequest` object (headers, query, body, etc.)     |
| `Response`    | The `HttpResponse` object (status code, headers, cookies) |
| `ModelState`  | Validation state of the model-bound data                  | 
| `User`        | The `ClaimsPrincipal` representing the authenticated user |
| `Url`         | URL generation helper                                     |

**Key Methods:**

| Method           | Status Code | Usage                                  |
| ---------------- | ----------- | -------------------------------------- |
| `Ok()`           | 200         | Successful response with optional body |
| `Created()`      | 201         | Resource successfully created          |
| `NoContent()`    | 204         | Success with no response body          |
| `BadRequest()`   | 400         | Client sent invalid data               |
| `Unauthorized()` | 401         | Authentication required                |
| `Forbid()`       | 403         | Authenticated but not authorized       |
| `NotFound()`     | 404         | Resource does not exist                | 
| `Conflict()`     | 409         | Request conflicts with current state   |
| `StatusCode()`   | Any         | Return any arbitrary status code       |

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var order = _orderService.Find(id);
        if (order is null)
            return NotFound();

        return Ok(order);
    }
}
```

### Controller (for MVC with Views)

`Controller` inherits everything from `ControllerBase` and adds view-rendering support. Use this when your actions return HTML pages via Razor views (see [[17.07 - Views and Razor]]).

**Additional Properties:**

| Property | Description |
|---|---|
| `ViewData` | Dictionary for passing data to views |
| `ViewBag` | Dynamic wrapper around `ViewData` |
| `TempData` | Data that survives a single redirect |

**Additional Methods:**

| Method | Description |
|---|---|
| `View()` | Renders a Razor view |
| `PartialView()` | Renders a partial view |
| `Json()` | Serializes an object to JSON |
| `RedirectToAction()` | Redirects to another action |

```csharp
public class ProductsController : Controller
{
    [HttpGet]
    public IActionResult Index()
    {
        var products = _productService.GetAll();
        ViewBag.Title = "All Products";
        return View(products); // renders Views/Products/Index.cshtml
    }

    [HttpGet]
    public IActionResult Details(int id)
    {
        var product = _productService.Find(id);
        if (product is null)
            return NotFound();

        return View(product);
    }
}
```

```ad-tip
**Guideline:** Use `ControllerBase` for API controllers and `Controller` for MVC apps that render views. If you are building a pure API (which is the most common modern scenario), there is no reason to inherit from `Controller` and carry the view-rendering baggage.
```

---

## The [ApiController] Attribute

The `[ApiController]` attribute activates a set of opinionated conventions that make API development cleaner and more consistent. Apply it at the controller class level.

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // ...
}
```

### What It Enables

#### 1. Automatic Model Validation

Without `[ApiController]`, you must manually check `ModelState`:

```csharp
// WITHOUT [ApiController] -- manual validation
[HttpPost]
public IActionResult Create(ProductCreateDto dto)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    var product = _productService.Create(dto);
    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
}
```

With `[ApiController]`, the framework does this automatically. If `ModelState` is invalid, it short-circuits and returns a 400 with `ProblemDetails` before your action code ever runs:

```csharp
// WITH [ApiController] -- no manual check needed
[HttpPost]
public IActionResult Create(ProductCreateDto dto)
{
    // If we reach here, ModelState is guaranteed valid
    var product = _productService.Create(dto);
    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
}
```

See [[Validation]] for details on data annotations and custom validators.

#### 2. Binding Source Inference

The framework infers where to bind action parameters from:

| Parameter Type | Inferred Source |
|---|---|
| Complex types (classes) | `[FromBody]` |
| `IFormFile` / `IFormFileCollection` | `[FromForm]` |
| Route parameter match | `[FromRoute]` |
| Everything else | `[FromQuery]` |

Without `[ApiController]`, you would need to explicitly decorate every parameter. See [[Model Binding]] for full details.

#### 3. ProblemDetails Error Responses

Error responses automatically follow the **RFC 7807** `ProblemDetails` format:

```json
{
    "type": "https://tools.ietf.org/html/rfc7807",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "traceId": "00-abc123...",
    "errors": {
        "Name": ["The Name field is required."],
        "Price": ["The field Price must be between 0.01 and 10000."]
    }
}
```

#### 4. Attribute Routing Required

When `[ApiController]` is applied, **convention-based routing does not work**. You must use attribute routing (`[Route]`, `[HttpGet]`, etc.). This is intentional -- API endpoints should have explicit, predictable URLs. See [[17.05 - Routing]].

```ad-warning
If you forget to add `[Route]` to an `[ApiController]` class, the app will throw an exception at startup telling you that attribute routing is required.
```

---

## Controller Naming Convention

ASP.NET Core strips the `Controller` suffix from the class name when generating route tokens.

| Class Name | Route Token `[controller]` | Default URL |
|---|---|---|
| `ProductsController` | `Products` | `/Products` or `/api/Products` |
| `HomeController` | `Home` | `/Home` |
| `OrderItemsController` | `OrderItems` | `/OrderItems` |

### Convention-Based Routing (MVC)

Used primarily for MVC apps that serve views:

```csharp
// Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

This maps `GET /Products/Details/5` to `ProductsController.Details(5)`.

### Attribute Routing (APIs)

Preferred for APIs. The `[controller]` token is replaced with the class name minus the `Controller` suffix:

```csharp
[ApiController]
[Route("api/[controller]")]  // resolves to "api/Products"
public class ProductsController : ControllerBase
{
    [HttpGet]           // GET api/Products
    public IActionResult GetAll() { ... }

    [HttpGet("{id}")]   // GET api/Products/5
    public IActionResult GetById(int id) { ... }

    [HttpPost]          // POST api/Products
    public IActionResult Create(ProductCreateDto dto) { ... }
}
```

See [[17.05 - Routing]] for a deep dive on routing.

---

## Constructor Injection

Controllers receive their dependencies through constructor injection from the built-in DI container. This is the standard way to get services into a controller.

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductRepository _productRepo;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(
        IProductRepository productRepo,
        ILogger<ProductsController> logger)
    {
        _productRepo = productRepo;
        _logger = logger;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id)
    {
        _logger.LogInformation("Fetching product {ProductId}", id);
        var product = await _productRepo.GetByIdAsync(id);

        if (product is null)
        {
            _logger.LogWarning("Product {ProductId} not found", id);
            return NotFound();
        }

        return Ok(product);
    }
}
```

```ad-info
Controllers have a **transient** lifetime -- a new instance is created for every HTTP request. This means constructor parameters are resolved fresh each time, which is safe even for scoped services like `DbContext`.
```

You can also inject services into individual actions using the `[FromServices]` attribute, which is useful for dependencies needed by only one action:

```csharp
[HttpPost("export")]
public async Task<IActionResult> Export(
    [FromServices] IExportService exportService)
{
    var data = await exportService.GenerateReportAsync();
    return File(data, "application/pdf", "report.pdf");
}
```

See [[17.03 - Dependency Injection]] for full coverage of the DI container, lifetimes, and registration patterns.

---

## The Request Lifecycle

Understanding the pipeline a request travels through is essential. Here is the order of operations when a request hits a controller action:

```
Client sends HTTP request
       |
       v
1. Routing selects controller + action         --> see [[17.05 - Routing]]
       |
       v
2. Controller is instantiated (DI resolves
   constructor parameters)                     --> see [[17.03 - Dependency Injection]]
       |
       v
3. Model binding maps request data
   (route, query, body, headers, form)
   to action parameters                        --> see [[Model Binding]]
       |
       v
4. Filters execute (authorization, resource,
   action, exception filters)                  --> see [[Filters]]
       |
       v
5. Action method executes
       |
       v
6. Action result executes
   (serializes response body, sets status)     --> see [[Action Results]]
       |
       v
7. Response sent to client
```

```ad-note
If model validation fails (with `[ApiController]`), the pipeline short-circuits at step 3 and returns a 400 response. The action method never executes.
```

---

## Async Actions

ASP.NET Core is built on an asynchronous pipeline. Actions that perform I/O (database calls, HTTP calls, file reads) should **always** be async.

### Sync vs Async

```csharp
// BAD -- blocks a thread pool thread while waiting for the database
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var product = _productRepo.GetById(id);  // synchronous -- thread is blocked
    if (product is null)
        return NotFound();

    return Ok(product);
}

// GOOD -- frees the thread while waiting for the database
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id)
{
    var product = await _productRepo.GetByIdAsync(id);  // thread returns to pool
    if (product is null)
        return NotFound();

    return Ok(product);
}
```

```ad-danger
**Never use `Task.Run()` in ASP.NET Core actions.** ASP.NET Core shares its thread pool with the request pipeline. Wrapping synchronous code in `Task.Run()` does not help -- it just offloads work to another thread pool thread while the original thread waits. You end up using **two** threads instead of one, reducing overall throughput.
```

### Cancellation Tokens

ASP.NET Core automatically binds a `CancellationToken` parameter to the request's abort token. If the client disconnects, the token is cancelled, and your async operation can stop early:

```csharp
[HttpGet]
public async Task<IActionResult> GetAll(CancellationToken cancellationToken)
{
    // If the client disconnects, this query is cancelled
    var products = await _dbContext.Products
        .AsNoTracking()
        .ToListAsync(cancellationToken);

    return Ok(products);
}
```

```ad-tip
Always pass the `CancellationToken` down to every async call in the chain (EF Core queries, `HttpClient` calls, etc.). This ensures resources are freed promptly when clients abandon requests.
```

### Strongly-Typed Return Values

Instead of `Task<IActionResult>`, you can use `Task<ActionResult<T>>` for better Swagger/OpenAPI documentation:

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<ProductDto>> GetById(int id)
{
    var product = await _productRepo.GetByIdAsync(id);
    if (product is null)
        return NotFound();  // implicitly converts to ActionResult<ProductDto>

    return product;  // implicitly wraps in Ok(product)
}
```

---

## HttpContext, Request, Response

Every controller has access to the full HTTP context through inherited properties.

### HttpContext

The `HttpContext` object is the container for everything related to the current HTTP request and response. It is available as `this.HttpContext` inside any controller.

### Request

The `Request` property (`HttpRequest`) gives access to all incoming data:

```csharp
[HttpGet]
public IActionResult Inspect()
{
    var method = Request.Method;               // "GET"
    var path = Request.Path;                   // "/api/products"
    var queryString = Request.QueryString;     // "?page=2&size=10"
    var host = Request.Host;                   // "localhost:5001"
    var scheme = Request.Scheme;               // "https"
    var contentType = Request.ContentType;     // "application/json"
    var isHttps = Request.IsHttps;             // true

    // Query parameters
    var page = Request.Query["page"];          // "2"

    // Headers
    var userAgent = Request.Headers["User-Agent"];
    var correlationId = Request.Headers["X-Correlation-Id"];

    return Ok(new { method, path, host, correlationId = correlationId.ToString() });
}
```

### Response

The `Response` property (`HttpResponse`) lets you modify the outgoing response:

```csharp
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id)
{
    var product = await _productRepo.GetByIdAsync(id);
    if (product is null)
        return NotFound();

    // Add custom response headers
    Response.Headers.Append("X-Product-Version", product.Version.ToString());
    Response.Headers.Append("Cache-Control", "public, max-age=60");

    return Ok(product);
}
```

### User

The `User` property is a `ClaimsPrincipal` representing the authenticated user:

```csharp
[Authorize]
[HttpGet("profile")]
public IActionResult GetProfile()
{
    var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
    var email = User.FindFirstValue(ClaimTypes.Email);
    var roles = User.FindAll(ClaimTypes.Role).Select(c => c.Value);
    var isAdmin = User.IsInRole("Admin");

    return Ok(new { userId, email, roles, isAdmin });
}
```

### Reading a Custom Header -- Practical Example

```csharp
[HttpPost]
public async Task<IActionResult> ProcessWebhook()
{
    // Verify the webhook signature from a custom header
    if (!Request.Headers.TryGetValue("X-Webhook-Signature", out var signature))
    {
        return BadRequest("Missing webhook signature header");
    }

    using var reader = new StreamReader(Request.Body);
    var body = await reader.ReadToEndAsync();

    if (!_webhookService.VerifySignature(body, signature!))
    {
        return Unauthorized("Invalid webhook signature");
    }

    await _webhookService.ProcessAsync(body);
    return Ok();
}
```

---

## Areas

Areas provide a way to organize a large MVC application into smaller functional groupings. Each area has its own set of controllers, views, and models.

### Setting Up an Area

Decorate the controller with the `[Area]` attribute:

```csharp
[Area("Admin")]
public class DashboardController : Controller
{
    public IActionResult Index()
    {
        return View(); // looks in Areas/Admin/Views/Dashboard/Index.cshtml
    }
}

[Area("Admin")]
public class UsersController : Controller
{
    public IActionResult Index()
    {
        return View(); // looks in Areas/Admin/Views/Users/Index.cshtml
    }
}
```

### Folder Structure

```
Project/
  Areas/
    Admin/
      Controllers/
        DashboardController.cs
        UsersController.cs
      Views/
        Dashboard/
          Index.cshtml
        Users/
          Index.cshtml
    Customer/
      Controllers/
        OrdersController.cs
      Views/
        Orders/
          Index.cshtml
```

### Route Configuration

Add an area-aware route **before** the default route:

```csharp
// Program.cs
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

The `{area:exists}` route constraint ensures the segment only matches if an area with that name actually exists.

```ad-info
Areas are primarily used in MVC apps with views. For APIs, it is more common to organize controllers by feature folders or use route prefixes instead of areas.
```

---

## Real-World Example

A complete `ProductsController` for an e-commerce API demonstrating all the concepts covered above.

### The DTO Classes

```csharp
public record ProductDto(
    int Id,
    string Name,
    string Description,
    decimal Price,
    int StockQuantity,
    string Category);

public record ProductCreateDto(
    [Required] [StringLength(200)] string Name,
    [StringLength(2000)] string Description,
    [Range(0.01, 100_000)] decimal Price,
    [Range(0, int.MaxValue)] int StockQuantity,
    [Required] string Category);

public record ProductUpdateDto(
    [Required] [StringLength(200)] string Name,
    [StringLength(2000)] string Description,
    [Range(0.01, 100_000)] decimal Price,
    [Range(0, int.MaxValue)] int StockQuantity,
    [Required] string Category);
```

### The Controller

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    private readonly IProductRepository _productRepo;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(
        IProductRepository productRepo,
        ILogger<ProductsController> logger)
    {
        _productRepo = productRepo;
        _logger = logger;
    }

    /// <summary>
    /// Returns all products, optionally filtered by category.
    /// </summary>
    [HttpGet]
    [ProducesResponseType(typeof(IEnumerable<ProductDto>), StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<ProductDto>>> GetAll(
        [FromQuery] string? category,
        CancellationToken cancellationToken)
    {
        _logger.LogInformation("Fetching products. Category filter: {Category}", category);

        var products = category is not null
            ? await _productRepo.GetByCategoryAsync(category, cancellationToken)
            : await _productRepo.GetAllAsync(cancellationToken);

        return Ok(products);
    }

    /// <summary>
    /// Returns a single product by its ID.
    /// </summary>
    [HttpGet("{id:int}")]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductDto>> GetById(
        int id,
        CancellationToken cancellationToken)
    {
        var product = await _productRepo.GetByIdAsync(id, cancellationToken);

        if (product is null)
        {
            _logger.LogWarning("Product {ProductId} not found", id);
            return NotFound();
        }

        return Ok(product);
    }

    /// <summary>
    /// Creates a new product.
    /// </summary>
    [HttpPost]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProductDto>> Create(
        ProductCreateDto dto,
        CancellationToken cancellationToken)
    {
        // [ApiController] guarantees ModelState is valid if we reach here

        _logger.LogInformation("Creating product: {ProductName}", dto.Name);

        var product = await _productRepo.CreateAsync(dto, cancellationToken);

        return CreatedAtAction(
            actionName: nameof(GetById),
            routeValues: new { id = product.Id },
            value: product);
    }

    /// <summary>
    /// Updates an existing product.
    /// </summary>
    [HttpPut("{id:int}")]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProductDto>> Update(
        int id,
        ProductUpdateDto dto,
        CancellationToken cancellationToken)
    {
        var existing = await _productRepo.GetByIdAsync(id, cancellationToken);

        if (existing is null)
        {
            _logger.LogWarning("Cannot update -- product {ProductId} not found", id);
            return NotFound();
        }

        _logger.LogInformation("Updating product {ProductId}", id);

        var updated = await _productRepo.UpdateAsync(id, dto, cancellationToken);
        return Ok(updated);
    }

    /// <summary>
    /// Deletes a product by its ID.
    /// </summary>
    [HttpDelete("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> Delete(
        int id,
        CancellationToken cancellationToken)
    {
        var existing = await _productRepo.GetByIdAsync(id, cancellationToken);

        if (existing is null)
        {
            _logger.LogWarning("Cannot delete -- product {ProductId} not found", id);
            return NotFound();
        }

        _logger.LogInformation("Deleting product {ProductId}", id);

        await _productRepo.DeleteAsync(id, cancellationToken);
        return NoContent();
    }
}
```

```ad-summary
**Key patterns demonstrated in the example above:**
- `ControllerBase` (not `Controller`) because this is a pure API
- `[ApiController]` for automatic validation, binding inference, and ProblemDetails
- `[Route("api/[controller]")]` for attribute-based routing
- Constructor injection for `IProductRepository` and `ILogger<T>`
- All actions are `async` with `CancellationToken`
- Strongly-typed `ActionResult<T>` for better OpenAPI documentation
- `[ProducesResponseType]` attributes for Swagger
- Proper status codes: 200, 201, 204, 400, 404
- `CreatedAtAction` returns 201 with a `Location` header pointing to the new resource
- Structured logging with message templates (not string interpolation)
```

---

## Related Notes

- [[Action Results]] -- all the `IActionResult` types and when to use each
- [[Model Binding]] -- how request data maps to action parameters
- [[Validation]] -- data annotations, `FluentValidation`, custom validators
- [[Filters]] -- authorization, action, exception, and result filters
- [[17.05 - Routing]] -- route templates, constraints, and attribute routing
- [[17.03 - Dependency Injection]] -- registering and resolving services
- [[17.07 - Views and Razor]] -- rendering HTML when using `Controller`
- [[17.08 - Web APIs]] -- building RESTful APIs with ASP.NET Core
