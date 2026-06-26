---
tags: [csharp, asp-net-core, web-api, controllers]
date: 2026-06-18
---

# API Controllers

API controllers are the backbone of building RESTful web services in ASP.NET Core. Unlike MVC controllers that return HTML views, ==API controllers return structured data== (JSON, XML, or other formats) that clients consume programmatically. This note covers everything from the fundamentals to production-quality patterns.

See also: [[Minimal APIs]] for the lightweight alternative, [[Content Negotiation]] for response format handling, and [[API Conventions]] for standardizing API behavior.

---

## Table of Contents

- [What Is an API Controller](#what-is-an-api-controller)
- [The ApiController Attribute](#the-apicontroller-attribute)
- [Inheriting from ControllerBase](#inheriting-from-controllerbase)
- [Routing for API Controllers](#routing-for-api-controllers)
- [RESTful Design and HTTP Verbs](#restful-design-and-http-verbs)
- [ActionResult and Return Types](#actionresult-and-return-types)
- [Data Transfer Objects (DTOs)](#data-transfer-objects-dtos)
- [Pagination](#pagination)
- [Filtering and Sorting](#filtering-and-sorting)
- [API Versioning](#api-versioning)
- [Real-World Example: Complete ProductsController](#real-world-example-complete-productscontroller)
- [Summary](#summary)

---

## What Is an API Controller

An **API controller** is a class that handles HTTP requests and returns data rather than HTML views. In a traditional MVC application, a controller action renders a Razor view. In an API controller, the action returns serialized data -- typically JSON -- that a frontend application, mobile app, or another service consumes.

The key distinction:

| Aspect | MVC Controller | API Controller |
|---|---|---|
| Returns | HTML views (Razor) | Data (JSON/XML) |
| Base class | `Controller` | `ControllerBase` |
| Content type | `text/html` | `application/json` |
| Consumers | Browsers | SPAs, mobile apps, services |
| View support | Yes | No |

A minimal API controller looks like this:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll()
    {
        var products = new[]
        {
            new { Id = 1, Name = "Keyboard", Price = 49.99m },
            new { Id = 2, Name = "Mouse", Price = 29.99m }
        };

        return Ok(products);
    }
}
```

When a client sends `GET /api/products`, ASP.NET Core:

1. Routes the request to `ProductsController.GetAll()`
2. Serializes the return value to JSON (by default via `System.Text.Json`)
3. Sends the response with `Content-Type: application/json` and status `200 OK`

> [!ad-note]
> ASP.NET Core uses **content negotiation** to determine the response format. If the client sends an `Accept: application/xml` header and XML formatters are configured, the framework can serialize to XML instead. See [[Content Negotiation]] for details.

> [!summary] Section Summary
> API controllers return structured data (JSON/XML) instead of HTML views. They inherit from `ControllerBase`, are decorated with `[ApiController]`, and serve as the foundation for RESTful web services consumed by SPAs, mobile apps, and other services.

---

## The ApiController Attribute

The **`[ApiController]`** attribute is applied to a controller class (or to the assembly) and enables several behaviors that are specifically designed for API development. ==This attribute is not strictly required, but you should always use it for API controllers== because it eliminates boilerplate and enforces best practices.

### What `[ApiController]` Enables

#### 1. Automatic HTTP 400 Responses for Invalid Models

Without `[ApiController]`, you must manually check `ModelState.IsValid`:

```csharp
// WITHOUT [ApiController] -- manual validation
[HttpPost]
public IActionResult Create(CreateProductDto dto)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    // ... process the request
}
```

With `[ApiController]`, invalid model state ==automatically returns a 400 Bad Request== before your action method even executes:

```csharp
// WITH [ApiController] -- automatic validation
[HttpPost]
public IActionResult Create(CreateProductDto dto)
{
    // ModelState is guaranteed to be valid here
    // No need for manual checking
    // ... process the request
}
```

The automatic response uses the **ProblemDetails** format:

```json
{
    "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "errors": {
        "Name": ["The Name field is required."],
        "Price": ["The field Price must be between 0.01 and 99999."]
    }
}
```

> [!tip]
> You can customize the automatic 400 response by configuring `ApiBehaviorOptions` in `Program.cs`:
> ```csharp
> builder.Services.Configure<ApiBehaviorOptions>(options =>
> {
>     options.InvalidModelStateResponseFactory = context =>
>     {
>         var errors = context.ModelState
>             .Where(e => e.Value?.Errors.Count > 0)
>             .ToDictionary(
>                 kvp => kvp.Key,
>                 kvp => kvp.Value!.Errors.Select(e => e.ErrorMessage).ToArray()
>             );
>
>         return new BadRequestObjectResult(new
>         {
>             Message = "Validation failed",
>             Errors = errors
>         });
>     };
> });
> ```

#### 2. Binding Source Parameter Inference

`[ApiController]` infers where action parameters come from based on their type:

| Parameter Type | Inferred Source | Attribute |
|---|---|---|
| Complex types (classes) | Request body | `[FromBody]` |
| `IFormFile`, `IFormFileCollection` | Form | `[FromForm]` |
| Parameters matching route template | Route | `[FromRoute]` |
| All other simple types | Query string | `[FromQuery]` |
| Services registered in DI | Services | `[FromServices]` |

```csharp
[HttpPut("{id}")]
public IActionResult Update(
    int id,                   // [FromRoute] inferred (matches {id} in route)
    UpdateProductDto dto)     // [FromBody] inferred (complex type)
{
    // No explicit [FromRoute] or [FromBody] needed
}

[HttpGet]
public IActionResult Search(
    string? name,             // [FromQuery] inferred (simple type, no route match)
    decimal? minPrice)        // [FromQuery] inferred
{
    // Query string: ?name=keyboard&minPrice=10
}
```

> [!warning]
> Binding source inference can be overridden with explicit attributes. This is necessary when the inferred source is wrong -- for example, receiving a simple type from the body:
> ```csharp
> [HttpPost("set-quantity")]
> public IActionResult SetQuantity([FromBody] int quantity)
> {
>     // Without [FromBody], 'int' would be inferred as [FromQuery]
> }
> ```

#### 3. ProblemDetails for Error Status Codes

When `[ApiController]` is applied and you return an error status code (4xx or 5xx) without a body, ==the framework automatically wraps it in a ProblemDetails response==:

```csharp
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var product = _repository.Find(id);
    if (product is null)
        return NotFound();  // Automatically wrapped in ProblemDetails
    
    return Ok(product);
}
```

The `NotFound()` call produces:

```json
{
    "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
    "title": "Not Found",
    "status": 404
}
```

#### 4. Multipart/Form-Data Request Inference

When an action parameter is annotated with `[FromForm]` or is of type `IFormFile`, the `[ApiController]` attribute infers the `multipart/form-data` content type for that request.

### Applying `[ApiController]` at the Assembly Level

Instead of decorating every controller individually, you can apply it once at the assembly level in `Program.cs` or any file:

```csharp
[assembly: ApiController]

// Now all controllers in this assembly behave as API controllers
```

> [!summary] Section Summary
> The `[ApiController]` attribute enables automatic model validation (400 responses), binding source inference (eliminating explicit `[FromBody]`/`[FromRoute]` attributes), and ProblemDetails responses for error status codes. Always apply it to API controllers.

---

## Inheriting from ControllerBase

API controllers should inherit from **`ControllerBase`**, not from `Controller`.

```
System.Object
  -> ControllerBase      <-- Use this for APIs
      -> Controller      <-- Use this for MVC (adds View support)
```

**`ControllerBase`** provides everything an API controller needs:

- `Ok()`, `Ok(value)` -- 200 OK
- `Created()`, `CreatedAtAction()`, `CreatedAtRoute()` -- 201 Created
- `NoContent()` -- 204 No Content
- `BadRequest()`, `BadRequest(error)` -- 400 Bad Request
- `Unauthorized()` -- 401 Unauthorized
- `Forbid()` -- 403 Forbidden
- `NotFound()`, `NotFound(value)` -- 404 Not Found
- `Conflict()`, `Conflict(error)` -- 409 Conflict
- `UnprocessableEntity()` -- 422 Unprocessable Entity
- `StatusCode(int)` -- Any status code
- `File()` -- File responses
- `HttpContext`, `Request`, `Response` -- Access to the request pipeline
- `ModelState` -- Model validation state
- `User` -- The authenticated user's claims

**`Controller`** adds view-related members like `View()`, `PartialView()`, `ViewData`, `ViewBag`, and `TempData`. Since API controllers never render views, using `Controller` adds unnecessary overhead and can be misleading.

> [!danger]
> Inheriting from `Controller` instead of `ControllerBase` for an API controller is a common beginner mistake. It works, but it pollutes your controller with view-related methods and properties that have no purpose in an API context. Always use `ControllerBase` for API controllers.

```csharp
// CORRECT for API controllers
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    // API actions here
}

// INCORRECT for API controllers -- unnecessary view support
[ApiController]
[Route("api/[controller]")]
public class OrdersController : Controller  // Don't do this
{
    // View(), ViewBag, TempData are all available but useless
}
```

> [!summary] Section Summary
> Always inherit from `ControllerBase` for API controllers, not `Controller`. `ControllerBase` provides all the HTTP response helper methods (`Ok()`, `NotFound()`, `BadRequest()`, etc.) without the unnecessary view-rendering baggage that `Controller` adds.

---

## Routing for API Controllers

**Attribute routing** is the standard approach for API controllers. Convention-based routing (used in MVC) is not recommended for APIs.

### The Standard Route Template

```csharp
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // [controller] is replaced with "products" (class name minus "Controller" suffix)
}
```

The **`[controller]`** token is a route token replacement. For `ProductsController`, it becomes `products`. For `OrderItemsController`, it becomes `orderitems`.

### Action-Level Route Templates

Each action method specifies its HTTP verb and optional route extension:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]                  // GET api/products
    public IActionResult GetAll() { ... }

    [HttpGet("{id}")]          // GET api/products/5
    public IActionResult GetById(int id) { ... }

    [HttpGet("{id}/reviews")]  // GET api/products/5/reviews
    public IActionResult GetReviews(int id) { ... }

    [HttpPost]                 // POST api/products
    public IActionResult Create(CreateProductDto dto) { ... }

    [HttpPut("{id}")]          // PUT api/products/5
    public IActionResult Update(int id, UpdateProductDto dto) { ... }

    [HttpDelete("{id}")]       // DELETE api/products/5
    public IActionResult Delete(int id) { ... }
}
```

### Route Constraints

Route constraints restrict which values match a route parameter:

```csharp
[HttpGet("{id:int}")]                    // Only matches integer values
[HttpGet("{id:int:min(1)}")]             // Integer >= 1
[HttpGet("{slug:alpha}")]                // Only alphabetic characters
[HttpGet("{date:datetime}")]             // Valid DateTime values
[HttpGet("{id:guid}")]                   // Valid GUID values
[HttpGet("{name:minlength(3)}")]         // At least 3 characters
[HttpGet("{name:maxlength(50)}")]        // At most 50 characters
[HttpGet("{name:regex(^[a-z]+$)}")]      // Matches regex pattern
```

Multiple constraints can be combined:

```csharp
[HttpGet("{id:int:min(1):max(999)}")]    // Integer between 1 and 999
```

### Naming Routes for Link Generation

Named routes allow you to generate URLs to specific actions:

```csharp
[HttpGet("{id}", Name = "GetProductById")]
public IActionResult GetById(int id)
{
    var product = _repository.Find(id);
    if (product is null) return NotFound();
    return Ok(product);
}

[HttpPost]
public IActionResult Create(CreateProductDto dto)
{
    var product = _repository.Add(dto);
    
    // Generates a URL like "/api/products/42"
    return CreatedAtRoute("GetProductById", new { id = product.Id }, product);
}
```

> [!ad-note]
> You can also use `CreatedAtAction` which references the action method name instead of a route name:
> ```csharp
> return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
> ```

> [!summary] Section Summary
> API controllers use attribute routing with `[Route("api/[controller]")]` at the class level and HTTP verb attributes (`[HttpGet]`, `[HttpPost]`, etc.) at the action level. Route constraints validate parameters, and named routes enable URL generation for `CreatedAtRoute` responses.

---

## RESTful Design and HTTP Verbs

**REST (Representational State Transfer)** maps HTTP verbs to CRUD operations on resources. Each resource is identified by a URL, and the verb indicates the operation.

### The Standard Verb-to-CRUD Mapping

| HTTP Verb | CRUD Operation | Route Example | Description |
|---|---|---|---|
| `GET` | Read | `GET /api/products` | List all products |
| `GET` | Read | `GET /api/products/5` | Get product with ID 5 |
| `POST` | Create | `POST /api/products` | Create a new product |
| `PUT` | Update (full) | `PUT /api/products/5` | Replace product 5 entirely |
| `PATCH` | Update (partial) | `PATCH /api/products/5` | Update specific fields of product 5 |
| `DELETE` | Delete | `DELETE /api/products/5` | Delete product 5 |

### HTTP Response Codes by Verb

Each verb has expected response patterns:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // GET /api/products -> 200 OK with list
    [HttpGet]
    public ActionResult<IEnumerable<ProductDto>> GetAll()
    {
        var products = _repository.GetAll();
        return Ok(products);
    }

    // GET /api/products/5 -> 200 OK or 404 Not Found
    [HttpGet("{id}")]
    public ActionResult<ProductDto> GetById(int id)
    {
        var product = _repository.Find(id);
        if (product is null) return NotFound();
        return Ok(product);
    }

    // POST /api/products -> 201 Created with Location header
    [HttpPost]
    public ActionResult<ProductDto> Create(CreateProductDto dto)
    {
        var product = _repository.Add(dto);
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }

    // PUT /api/products/5 -> 204 No Content or 404 Not Found
    [HttpPut("{id}")]
    public IActionResult Update(int id, UpdateProductDto dto)
    {
        var existing = _repository.Find(id);
        if (existing is null) return NotFound();
        
        _repository.Update(id, dto);
        return NoContent();
    }

    // DELETE /api/products/5 -> 204 No Content or 404 Not Found
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        var existing = _repository.Find(id);
        if (existing is null) return NotFound();
        
        _repository.Delete(id);
        return NoContent();
    }
}
```

### PATCH with JSON Patch

**Partial updates** use the `PATCH` verb with a JSON Patch document. Install the `Microsoft.AspNetCore.JsonPatch` and `Microsoft.AspNetCore.Mvc.NewtonsoftJson` NuGet packages:

```bash
dotnet add package Microsoft.AspNetCore.JsonPatch
dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson
```

Configure Newtonsoft.Json (required for JSON Patch support):

```csharp
builder.Services
    .AddControllers()
    .AddNewtonsoftJson();
```

The PATCH action:

```csharp
[HttpPatch("{id}")]
public IActionResult Patch(int id, [FromBody] JsonPatchDocument<UpdateProductDto> patchDoc)
{
    var existing = _repository.Find(id);
    if (existing is null) return NotFound();

    var dto = MapToDto(existing);
    patchDoc.ApplyTo(dto, ModelState);

    if (!TryValidateModel(dto))
        return BadRequest(ModelState);

    _repository.Update(id, dto);
    return NoContent();
}
```

A JSON Patch request body looks like:

```json
[
    { "op": "replace", "path": "/price", "value": 59.99 },
    { "op": "replace", "path": "/name", "value": "Wireless Keyboard" }
]
```

> [!tip]
> The HTTP request for a PATCH operation uses `Content-Type: application/json-patch+json`, not `application/json`.

### Nested Resources

For resources that belong to another resource, use nested routes:

```csharp
[ApiController]
[Route("api/products/{productId}/reviews")]
public class ProductReviewsController : ControllerBase
{
    [HttpGet]                      // GET /api/products/5/reviews
    public IActionResult GetAll(int productId) { ... }

    [HttpGet("{reviewId}")]        // GET /api/products/5/reviews/3
    public IActionResult GetById(int productId, int reviewId) { ... }

    [HttpPost]                     // POST /api/products/5/reviews
    public IActionResult Create(int productId, CreateReviewDto dto) { ... }
}
```

### Idempotency

An important REST principle: ==`GET`, `PUT`, and `DELETE` should be idempotent== (calling them multiple times with the same input produces the same result). `POST` is not idempotent -- each call creates a new resource.

| Verb | Idempotent | Safe (read-only) |
|---|---|---|
| GET | Yes | Yes |
| POST | No | No |
| PUT | Yes | No |
| PATCH | No* | No |
| DELETE | Yes | No |

*PATCH can be idempotent depending on the operations in the patch document.

> [!summary] Section Summary
> RESTful APIs map HTTP verbs to CRUD operations: GET for reading, POST for creating, PUT for full updates, PATCH for partial updates, and DELETE for removal. Each verb has standard response codes. GET, PUT, and DELETE are idempotent. Nested routes handle sub-resources.

---

## ActionResult and Return Types

Choosing the right return type for your action methods affects both runtime behavior and API documentation (Swagger/OpenAPI).

### The Three Return Type Options

#### 1. Specific Type (Direct Return)

```csharp
[HttpGet]
public IEnumerable<ProductDto> GetAll()
{
    return _repository.GetAll();
}
```

- Always returns 200 OK
- Cannot return different status codes (404, 400, etc.)
- Limited Swagger documentation

#### 2. `IActionResult` (Maximum Flexibility)

```csharp
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var product = _repository.Find(id);
    if (product is null) return NotFound();
    return Ok(product);
}
```

- Can return any status code
- Swagger cannot infer the response type without explicit `[ProducesResponseType]`

#### 3. `ActionResult<T>` (Best of Both Worlds)

```csharp
[HttpGet("{id}")]
public ActionResult<ProductDto> GetById(int id)
{
    var product = _repository.Find(id);
    if (product is null) return NotFound();
    return product;  // Implicit conversion to ActionResult<ProductDto>
}
```

- ==This is the recommended return type for API actions==
- Swagger automatically infers the `200 OK` response type as `ProductDto`
- Can still return different status codes
- Supports implicit conversion from `T` (no need for `Ok(product)`)

### Documenting Response Types with `[ProducesResponseType]`

For complete OpenAPI documentation, annotate actions with the status codes they can return:

```csharp
[HttpGet("{id}")]
[ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public ActionResult<ProductDto> GetById(int id)
{
    var product = _repository.Find(id);
    if (product is null) return NotFound();
    return product;
}

[HttpPost]
[ProducesResponseType(typeof(ProductDto), StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public ActionResult<ProductDto> Create(CreateProductDto dto)
{
    var product = _repository.Add(dto);
    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
}
```

> [!tip]
> With `ActionResult<T>`, you can use the shorter form for the success response:
> ```csharp
> [ProducesResponseType(StatusCodes.Status200OK)]      // Type inferred from ActionResult<T>
> [ProducesResponseType(StatusCodes.Status404NotFound)]
> public ActionResult<ProductDto> GetById(int id) { ... }
> ```

### Controller-Level Response Type Annotations

Apply shared response types at the controller level with `[Produces]`:

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]                          // All actions produce JSON
[ProducesResponseType(StatusCodes.Status500InternalServerError)]  // All actions may 500
public class ProductsController : ControllerBase
{
    // Individual actions only need their specific annotations
}
```

See [[API Conventions]] for applying conventions across controllers.

> [!summary] Section Summary
> Use `ActionResult<T>` as the return type for API actions -- it combines type safety, flexible status code returns, and automatic Swagger/OpenAPI documentation. Annotate actions with `[ProducesResponseType]` to document all possible response codes.

---

## Data Transfer Objects (DTOs)

A **Data Transfer Object (DTO)** is a simple class that defines the shape of data sent to or from an API. ==Never expose your entity/domain models directly through API endpoints.==

### Why Use DTOs?

Consider this entity model:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public decimal CostPrice { get; set; }          // Sensitive! Don't expose
    public string InternalSku { get; set; } = "";   // Internal! Don't expose
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
    public bool IsDeleted { get; set; }              // Soft delete flag
    
    public int CategoryId { get; set; }
    public Category Category { get; set; } = null!;  // Navigation property
    public ICollection<Review> Reviews { get; set; } = new List<Review>();
}
```

Problems with exposing this directly:

1. **Security**: `CostPrice` and `InternalSku` are internal data
2. **Over-posting**: A malicious client could set `IsDeleted = true` or `CostPrice = 0`
3. **Circular references**: `Product -> Category -> Products -> ...` causes serialization errors
4. **API contract coupling**: Changing the entity breaks the API
5. **Validation concerns**: Entity validation rules differ from input validation

### Input DTOs vs Output DTOs

Separate DTOs for input (what the client sends) and output (what the API returns):

```csharp
// OUTPUT DTO -- what the API returns to clients
public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public string CategoryName { get; set; } = string.Empty;
    public double AverageRating { get; set; }
    public int ReviewCount { get; set; }
}

// INPUT DTO for creation -- what the client sends to create
public class CreateProductDto
{
    [Required]
    [StringLength(200, MinimumLength = 1)]
    public string Name { get; set; } = string.Empty;

    [StringLength(2000)]
    public string Description { get; set; } = string.Empty;

    [Required]
    [Range(0.01, 99999.99)]
    public decimal Price { get; set; }

    [Required]
    public int CategoryId { get; set; }
}

// INPUT DTO for updating -- fields may differ from creation
public class UpdateProductDto
{
    [Required]
    [StringLength(200, MinimumLength = 1)]
    public string Name { get; set; } = string.Empty;

    [StringLength(2000)]
    public string Description { get; set; } = string.Empty;

    [Required]
    [Range(0.01, 99999.99)]
    public decimal Price { get; set; }

    [Required]
    public int CategoryId { get; set; }
}
```

> [!ad-note]
> Input and update DTOs often look similar but may diverge over time. For example, `CreateProductDto` might require `CategoryId` while `UpdateProductDto` makes it optional. Keep them as separate classes to allow independent evolution.

### Mapping: Manual Methods

For small projects, manual mapping is straightforward and has zero overhead:

```csharp
public static class ProductMappingExtensions
{
    public static ProductDto ToDto(this Product product)
    {
        return new ProductDto
        {
            Id = product.Id,
            Name = product.Name,
            Description = product.Description,
            Price = product.Price,
            CategoryName = product.Category?.Name ?? string.Empty,
            AverageRating = product.Reviews.Any()
                ? product.Reviews.Average(r => r.Rating)
                : 0,
            ReviewCount = product.Reviews.Count
        };
    }

    public static Product ToEntity(this CreateProductDto dto)
    {
        return new Product
        {
            Name = dto.Name,
            Description = dto.Description,
            Price = dto.Price,
            CategoryId = dto.CategoryId,
            CreatedAt = DateTime.UtcNow
        };
    }

    public static void ApplyTo(this UpdateProductDto dto, Product product)
    {
        product.Name = dto.Name;
        product.Description = dto.Description;
        product.Price = dto.Price;
        product.CategoryId = dto.CategoryId;
        product.UpdatedAt = DateTime.UtcNow;
    }
}
```

Usage in the controller:

```csharp
[HttpGet("{id}")]
public ActionResult<ProductDto> GetById(int id)
{
    var product = _context.Products
        .Include(p => p.Category)
        .Include(p => p.Reviews)
        .FirstOrDefault(p => p.Id == id);
    
    if (product is null) return NotFound();
    
    return product.ToDto();
}

[HttpPost]
public ActionResult<ProductDto> Create(CreateProductDto dto)
{
    var product = dto.ToEntity();
    _context.Products.Add(product);
    _context.SaveChanges();
    
    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product.ToDto());
}
```

### Mapping: AutoMapper

For larger projects with many entities, **AutoMapper** reduces repetitive mapping code:

```bash
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```

Define mapping profiles:

```csharp
public class ProductMappingProfile : Profile
{
    public ProductMappingProfile()
    {
        CreateMap<Product, ProductDto>()
            .ForMember(dest => dest.CategoryName, 
                       opt => opt.MapFrom(src => src.Category.Name))
            .ForMember(dest => dest.AverageRating,
                       opt => opt.MapFrom(src => src.Reviews.Average(r => r.Rating)))
            .ForMember(dest => dest.ReviewCount,
                       opt => opt.MapFrom(src => src.Reviews.Count));

        CreateMap<CreateProductDto, Product>()
            .ForMember(dest => dest.CreatedAt, opt => opt.MapFrom(_ => DateTime.UtcNow));

        CreateMap<UpdateProductDto, Product>()
            .ForMember(dest => dest.UpdatedAt, opt => opt.MapFrom(_ => DateTime.UtcNow));
    }
}
```

Register in `Program.cs`:

```csharp
builder.Services.AddAutoMapper(typeof(Program).Assembly);
```

Use in the controller:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly AppDbContext _context;
    private readonly IMapper _mapper;

    public ProductsController(AppDbContext context, IMapper mapper)
    {
        _context = context;
        _mapper = mapper;
    }

    [HttpGet("{id}")]
    public ActionResult<ProductDto> GetById(int id)
    {
        var product = _context.Products
            .Include(p => p.Category)
            .Include(p => p.Reviews)
            .FirstOrDefault(p => p.Id == id);
        
        if (product is null) return NotFound();
        
        return _mapper.Map<ProductDto>(product);
    }
}
```

> [!warning]
> AutoMapper adds a runtime dependency and can hide bugs where properties silently don't map. For performance-critical paths or small projects, manual mapping is often the better choice. If you use AutoMapper, always write unit tests to verify your mapping profiles.

> [!summary] Section Summary
> DTOs decouple your API contract from your domain model, preventing over-posting attacks, hiding sensitive data, and avoiding serialization issues. Use separate input and output DTOs. Map between entities and DTOs using manual extension methods (simple, explicit) or AutoMapper (convenient for large projects).

---

## Pagination

Production APIs must paginate list endpoints. Returning thousands of records in a single response wastes bandwidth, memory, and time. ==Always paginate collection endpoints.==

### Query Parameters for Pagination

The standard approach uses `page` (or `pageNumber`) and `pageSize` query parameters:

```http
GET /api/products?page=2&pageSize=25
```

### Pagination Models

```csharp
// Request model for pagination parameters
public class PaginationParams
{
    private const int MaxPageSize = 100;
    private int _pageSize = 10;

    public int Page { get; set; } = 1;
    
    public int PageSize
    {
        get => _pageSize;
        set => _pageSize = value > MaxPageSize ? MaxPageSize : value;
    }
}

// Response wrapper with pagination metadata
public class PagedResult<T>
{
    public IEnumerable<T> Items { get; set; } = Enumerable.Empty<T>();
    public int Page { get; set; }
    public int PageSize { get; set; }
    public int TotalCount { get; set; }
    public int TotalPages => (int)Math.Ceiling(TotalCount / (double)PageSize);
    public bool HasPreviousPage => Page > 1;
    public bool HasNextPage => Page < TotalPages;
}
```

### Implementing Pagination

```csharp
[HttpGet]
[ProducesResponseType(typeof(PagedResult<ProductDto>), StatusCodes.Status200OK)]
public ActionResult<PagedResult<ProductDto>> GetAll(
    [FromQuery] PaginationParams paginationParams)
{
    var query = _context.Products
        .Include(p => p.Category)
        .AsNoTracking();

    var totalCount = query.Count();

    var products = query
        .OrderBy(p => p.Id)
        .Skip((paginationParams.Page - 1) * paginationParams.PageSize)
        .Take(paginationParams.PageSize)
        .Select(p => new ProductDto
        {
            Id = p.Id,
            Name = p.Name,
            Description = p.Description,
            Price = p.Price,
            CategoryName = p.Category.Name
        })
        .ToList();

    var result = new PagedResult<ProductDto>
    {
        Items = products,
        Page = paginationParams.Page,
        PageSize = paginationParams.PageSize,
        TotalCount = totalCount
    };

    return Ok(result);
}
```

The response looks like:

```json
{
    "items": [
        { "id": 26, "name": "USB Hub", "price": 24.99, "categoryName": "Accessories" },
        { "id": 27, "name": "Webcam", "price": 79.99, "categoryName": "Peripherals" }
    ],
    "page": 2,
    "pageSize": 25,
    "totalCount": 142,
    "totalPages": 6,
    "hasPreviousPage": true,
    "hasNextPage": true
}
```

### Adding Pagination Headers

Some APIs return pagination metadata in response headers instead of (or in addition to) the body:

```csharp
[HttpGet]
public ActionResult<IEnumerable<ProductDto>> GetAll(
    [FromQuery] PaginationParams paginationParams)
{
    var query = _context.Products.AsNoTracking();
    var totalCount = query.Count();
    var totalPages = (int)Math.Ceiling(totalCount / (double)paginationParams.PageSize);

    Response.Headers.Append("X-Total-Count", totalCount.ToString());
    Response.Headers.Append("X-Total-Pages", totalPages.ToString());
    Response.Headers.Append("X-Page", paginationParams.Page.ToString());
    Response.Headers.Append("X-Page-Size", paginationParams.PageSize.ToString());

    var products = query
        .OrderBy(p => p.Id)
        .Skip((paginationParams.Page - 1) * paginationParams.PageSize)
        .Take(paginationParams.PageSize)
        .Select(p => p.ToDto())
        .ToList();

    return Ok(products);
}
```

> [!tip]
> If you use custom headers, remember to expose them in your CORS policy:
> ```csharp
> builder.Services.AddCors(options =>
> {
>     options.AddDefaultPolicy(policy =>
>     {
>         policy.WithExposedHeaders(
>             "X-Total-Count", "X-Total-Pages", "X-Page", "X-Page-Size");
>     });
> });
> ```

### Reusable Pagination Extension Method

Create an extension method to apply pagination to any `IQueryable<T>`:

```csharp
public static class QueryableExtensions
{
    public static PagedResult<T> ToPagedResult<T>(
        this IQueryable<T> query, 
        int page, 
        int pageSize)
    {
        var totalCount = query.Count();

        var items = query
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .ToList();

        return new PagedResult<T>
        {
            Items = items,
            Page = page,
            PageSize = pageSize,
            TotalCount = totalCount
        };
    }
}
```

Usage:

```csharp
[HttpGet]
public ActionResult<PagedResult<ProductDto>> GetAll([FromQuery] PaginationParams p)
{
    var result = _context.Products
        .AsNoTracking()
        .OrderBy(x => x.Id)
        .Select(x => x.ToDto())
        .ToPagedResult(p.Page, p.PageSize);

    return Ok(result);
}
```

> [!summary] Section Summary
> Always paginate collection endpoints using `page` and `pageSize` query parameters. Cap the maximum page size to prevent abuse. Return pagination metadata (total count, total pages, has next/previous) either in the response body or via custom headers. Use reusable extension methods to keep pagination logic DRY.

---

## Filtering and Sorting

Production APIs need more than just pagination. Clients need to **filter** data by criteria and **sort** results by specific fields.

### Filtering with Query Parameters

Define a filter model that combines pagination with filtering:

```csharp
public class ProductFilterParams : PaginationParams
{
    public string? Name { get; set; }
    public int? CategoryId { get; set; }
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
    public bool? InStock { get; set; }
}
```

Apply filters conditionally:

```csharp
[HttpGet]
public ActionResult<PagedResult<ProductDto>> GetAll(
    [FromQuery] ProductFilterParams filter)
{
    var query = _context.Products
        .Include(p => p.Category)
        .AsNoTracking();

    // Apply filters conditionally
    if (!string.IsNullOrWhiteSpace(filter.Name))
        query = query.Where(p => p.Name.Contains(filter.Name));

    if (filter.CategoryId.HasValue)
        query = query.Where(p => p.CategoryId == filter.CategoryId.Value);

    if (filter.MinPrice.HasValue)
        query = query.Where(p => p.Price >= filter.MinPrice.Value);

    if (filter.MaxPrice.HasValue)
        query = query.Where(p => p.Price <= filter.MaxPrice.Value);

    if (filter.InStock.HasValue)
        query = query.Where(p => p.StockQuantity > 0 == filter.InStock.Value);

    var result = query
        .OrderBy(p => p.Name)
        .Select(p => new ProductDto
        {
            Id = p.Id,
            Name = p.Name,
            Price = p.Price,
            CategoryName = p.Category.Name
        })
        .ToPagedResult(filter.Page, filter.PageSize);

    return Ok(result);
}
```

The request:

```http
GET /api/products?name=key&categoryId=3&minPrice=10&maxPrice=100&page=1&pageSize=20
```

### Sorting with Query Parameters

Allow clients to specify sort field and direction:

```csharp
public class ProductFilterParams : PaginationParams
{
    // ... filter properties ...

    public string? SortBy { get; set; }          // Field name: "name", "price", "date"
    public string SortOrder { get; set; } = "asc"; // "asc" or "desc"
}
```

Implement sorting with a safe whitelist approach:

```csharp
private static IQueryable<Product> ApplySorting(
    IQueryable<Product> query,
    string? sortBy,
    string sortOrder)
{
    var isDescending = sortOrder.Equals("desc", StringComparison.OrdinalIgnoreCase);

    return sortBy?.ToLower() switch
    {
        "name"     => isDescending ? query.OrderByDescending(p => p.Name)
                                   : query.OrderBy(p => p.Name),
        "price"    => isDescending ? query.OrderByDescending(p => p.Price)
                                   : query.OrderBy(p => p.Price),
        "date"     => isDescending ? query.OrderByDescending(p => p.CreatedAt)
                                   : query.OrderBy(p => p.CreatedAt),
        "category" => isDescending ? query.OrderByDescending(p => p.Category.Name)
                                   : query.OrderBy(p => p.Category.Name),
        _          => query.OrderBy(p => p.Id)  // Default sort
    };
}
```

Usage:

```http
GET /api/products?sortBy=price&sortOrder=desc&page=1&pageSize=20
```

> [!danger]
> Never pass user-provided sort field names directly into `OrderBy()` expressions using reflection or dynamic LINQ without validation. This opens the door to information disclosure or injection attacks. Always use a whitelist of allowed sort fields, as shown above.

### Search Endpoint

For more complex searching, create a dedicated search action:

```csharp
[HttpGet("search")]
public ActionResult<PagedResult<ProductDto>> Search(
    [FromQuery] string q,
    [FromQuery] PaginationParams pagination)
{
    if (string.IsNullOrWhiteSpace(q))
        return BadRequest(new { Message = "Search query 'q' is required." });

    var searchTerm = q.Trim().ToLower();

    var result = _context.Products
        .AsNoTracking()
        .Where(p => p.Name.ToLower().Contains(searchTerm)
                  || p.Description.ToLower().Contains(searchTerm)
                  || p.Category.Name.ToLower().Contains(searchTerm))
        .OrderBy(p => p.Name)
        .Select(p => p.ToDto())
        .ToPagedResult(pagination.Page, pagination.PageSize);

    return Ok(result);
}
```

```http
GET /api/products/search?q=wireless keyboard&page=1&pageSize=10
```

> [!summary] Section Summary
> Implement filtering by accepting nullable query parameters and building queries conditionally. Implement sorting with a whitelist of allowed fields to prevent injection. Combine filtering, sorting, and pagination into a single query parameter model for a flexible API.

---

## API Versioning

As your API evolves, you need to introduce breaking changes without breaking existing clients. **API versioning** allows multiple versions to coexist.

### Installing the Versioning Package

```bash
dotnet add package Asp.Versioning.Mvc
dotnet add package Asp.Versioning.Mvc.ApiExplorer
```

### Configuration in Program.cs

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;  // Adds api-supported-versions header
    
    // Choose one or combine multiple readers
    options.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),
        new QueryStringApiVersionReader("api-version"),
        new HeaderApiVersionReader("X-Api-Version")
    );
})
.AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});
```

### Approach 1: URL Path Versioning

==The most common and recommended approach.== The version is part of the URL path.

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0")]
public class ProductsV1Controller : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<ProductV1Dto>> GetAll()
    {
        // V1 implementation
    }
}

[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("2.0")]
public class ProductsV2Controller : ControllerBase
{
    [HttpGet]
    public ActionResult<IEnumerable<ProductV2Dto>> GetAll()
    {
        // V2 implementation with additional fields
    }
}
```

```http
GET /api/v1/products
GET /api/v2/products
```

### Approach 2: Query String Versioning

```http
GET /api/products?api-version=1.0
GET /api/products?api-version=2.0
```

### Approach 3: Header Versioning

```http
GET /api/products
X-Api-Version: 1.0
```

### Deprecating API Versions

Mark old versions as deprecated to signal clients:

```csharp
[ApiVersion("1.0", Deprecated = true)]  // Still works, but marked deprecated
[ApiVersion("2.0")]
[ApiController]
[Route("api/v{version:apiVersion}/products")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public ActionResult<IEnumerable<ProductV1Dto>> GetAllV1()
    {
        // Legacy implementation
    }

    [HttpGet]
    [MapToApiVersion("2.0")]
    public ActionResult<IEnumerable<ProductV2Dto>> GetAllV2()
    {
        // Current implementation
    }
}
```

The response includes deprecation headers:

```http
api-supported-versions: 1.0, 2.0
api-deprecated-versions: 1.0
```

### Comparison of Approaches

| Approach | Pros | Cons |
|---|---|---|
| URL path (`/v1/`) | Explicit, easy to test, cacheable | URL changes per version |
| Query string (`?api-version=1`) | URL stays the same | Easy to forget, not RESTful |
| Header (`X-Api-Version`) | Clean URLs | Hidden, harder to test in browser |

> [!tip]
> URL path versioning is the most widely used approach in the industry. APIs like GitHub, Stripe, and Twilio all use URL path versioning. Start with this unless you have a specific reason not to.

> [!summary] Section Summary
> API versioning lets you evolve your API without breaking existing clients. URL path versioning (`/api/v1/`, `/api/v2/`) is the most common and recommended approach. Use the `Asp.Versioning.Mvc` package to configure versioning, and mark old versions as deprecated to guide clients toward upgrading.

---

## Real-World Example: Complete ProductsController

Here is a production-quality `ProductsController` combining all concepts: CRUD operations, DTOs, pagination, filtering, sorting, proper status codes, and OpenAPI documentation.

### The Entity Model

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public decimal CostPrice { get; set; }
    public int StockQuantity { get; set; }
    public bool IsActive { get; set; } = true;
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
    
    public int CategoryId { get; set; }
    public Category Category { get; set; } = null!;
}
```

### The DTOs

```csharp
// Output DTO
public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public bool InStock => StockQuantity > 0;
    public string CategoryName { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; }
}

// Create input DTO
public class CreateProductDto
{
    [Required(ErrorMessage = "Product name is required.")]
    [StringLength(200, MinimumLength = 1, ErrorMessage = "Name must be between 1 and 200 characters.")]
    public string Name { get; set; } = string.Empty;

    [StringLength(2000, ErrorMessage = "Description cannot exceed 2000 characters.")]
    public string Description { get; set; } = string.Empty;

    [Required(ErrorMessage = "Price is required.")]
    [Range(0.01, 99999.99, ErrorMessage = "Price must be between 0.01 and 99999.99.")]
    public decimal Price { get; set; }

    [Range(0, int.MaxValue, ErrorMessage = "Stock quantity cannot be negative.")]
    public int StockQuantity { get; set; }

    [Required(ErrorMessage = "Category is required.")]
    public int CategoryId { get; set; }
}

// Update input DTO
public class UpdateProductDto
{
    [Required(ErrorMessage = "Product name is required.")]
    [StringLength(200, MinimumLength = 1)]
    public string Name { get; set; } = string.Empty;

    [StringLength(2000)]
    public string Description { get; set; } = string.Empty;

    [Required]
    [Range(0.01, 99999.99)]
    public decimal Price { get; set; }

    [Range(0, int.MaxValue)]
    public int StockQuantity { get; set; }

    [Required]
    public int CategoryId { get; set; }
}
```

### The Filter/Pagination Model

```csharp
public class ProductQueryParams
{
    private const int MaxPageSize = 50;
    private int _pageSize = 10;

    // Pagination
    public int Page { get; set; } = 1;
    public int PageSize
    {
        get => _pageSize;
        set => _pageSize = value > MaxPageSize ? MaxPageSize : (value < 1 ? 1 : value);
    }

    // Filtering
    public string? Search { get; set; }
    public int? CategoryId { get; set; }
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
    public bool? InStock { get; set; }

    // Sorting
    public string? SortBy { get; set; }
    public string SortOrder { get; set; } = "asc";
}
```

### The Complete Controller

```csharp
[ApiController]
[Route("api/[controller]")]
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    private readonly AppDbContext _context;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(AppDbContext context, ILogger<ProductsController> logger)
    {
        _context = context;
        _logger = logger;
    }

    /// <summary>
    /// Gets a paginated, filterable, sortable list of products.
    /// </summary>
    [HttpGet]
    [ProducesResponseType(typeof(PagedResult<ProductDto>), StatusCodes.Status200OK)]
    public ActionResult<PagedResult<ProductDto>> GetAll(
        [FromQuery] ProductQueryParams queryParams)
    {
        var query = _context.Products
            .Include(p => p.Category)
            .Where(p => p.IsActive)
            .AsNoTracking();

        // Apply filters
        if (!string.IsNullOrWhiteSpace(queryParams.Search))
        {
            var search = queryParams.Search.Trim().ToLower();
            query = query.Where(p =>
                p.Name.ToLower().Contains(search) ||
                p.Description.ToLower().Contains(search));
        }

        if (queryParams.CategoryId.HasValue)
            query = query.Where(p => p.CategoryId == queryParams.CategoryId.Value);

        if (queryParams.MinPrice.HasValue)
            query = query.Where(p => p.Price >= queryParams.MinPrice.Value);

        if (queryParams.MaxPrice.HasValue)
            query = query.Where(p => p.Price <= queryParams.MaxPrice.Value);

        if (queryParams.InStock.HasValue)
            query = query.Where(p => (p.StockQuantity > 0) == queryParams.InStock.Value);

        // Apply sorting
        query = ApplySorting(query, queryParams.SortBy, queryParams.SortOrder);

        // Apply pagination
        var totalCount = query.Count();
        var products = query
            .Skip((queryParams.Page - 1) * queryParams.PageSize)
            .Take(queryParams.PageSize)
            .Select(p => MapToDto(p))
            .ToList();

        var result = new PagedResult<ProductDto>
        {
            Items = products,
            Page = queryParams.Page,
            PageSize = queryParams.PageSize,
            TotalCount = totalCount
        };

        return Ok(result);
    }

    /// <summary>
    /// Gets a single product by its ID.
    /// </summary>
    [HttpGet("{id:int}", Name = "GetProductById")]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public ActionResult<ProductDto> GetById(int id)
    {
        var product = _context.Products
            .Include(p => p.Category)
            .AsNoTracking()
            .FirstOrDefault(p => p.Id == id && p.IsActive);

        if (product is null)
        {
            _logger.LogWarning("Product with ID {ProductId} not found.", id);
            return NotFound();
        }

        return MapToDto(product);
    }

    /// <summary>
    /// Creates a new product.
    /// </summary>
    [HttpPost]
    [ProducesResponseType(typeof(ProductDto), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public ActionResult<ProductDto> Create(CreateProductDto dto)
    {
        // Validate that the category exists
        var categoryExists = _context.Categories.Any(c => c.Id == dto.CategoryId);
        if (!categoryExists)
        {
            ModelState.AddModelError(nameof(dto.CategoryId), 
                $"Category with ID {dto.CategoryId} does not exist.");
            return ValidationProblem(ModelState);
        }

        var product = new Product
        {
            Name = dto.Name,
            Description = dto.Description,
            Price = dto.Price,
            StockQuantity = dto.StockQuantity,
            CategoryId = dto.CategoryId,
            CreatedAt = DateTime.UtcNow,
            IsActive = true
        };

        _context.Products.Add(product);
        _context.SaveChanges();

        // Reload with navigation properties for the response
        _context.Entry(product).Reference(p => p.Category).Load();

        _logger.LogInformation("Product created with ID {ProductId}.", product.Id);

        return CreatedAtRoute(
            "GetProductById",
            new { id = product.Id },
            MapToDto(product));
    }

    /// <summary>
    /// Fully updates an existing product.
    /// </summary>
    [HttpPut("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public IActionResult Update(int id, UpdateProductDto dto)
    {
        var product = _context.Products.Find(id);
        if (product is null || !product.IsActive)
            return NotFound();

        // Validate category
        var categoryExists = _context.Categories.Any(c => c.Id == dto.CategoryId);
        if (!categoryExists)
        {
            ModelState.AddModelError(nameof(dto.CategoryId),
                $"Category with ID {dto.CategoryId} does not exist.");
            return ValidationProblem(ModelState);
        }

        product.Name = dto.Name;
        product.Description = dto.Description;
        product.Price = dto.Price;
        product.StockQuantity = dto.StockQuantity;
        product.CategoryId = dto.CategoryId;
        product.UpdatedAt = DateTime.UtcNow;

        _context.SaveChanges();

        _logger.LogInformation("Product {ProductId} updated.", id);

        return NoContent();
    }

    /// <summary>
    /// Deletes a product (soft delete).
    /// </summary>
    [HttpDelete("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public IActionResult Delete(int id)
    {
        var product = _context.Products.Find(id);
        if (product is null || !product.IsActive)
            return NotFound();

        // Soft delete
        product.IsActive = false;
        product.UpdatedAt = DateTime.UtcNow;
        _context.SaveChanges();

        _logger.LogInformation("Product {ProductId} soft-deleted.", id);

        return NoContent();
    }

    // ---- Private helpers ----

    private static ProductDto MapToDto(Product product)
    {
        return new ProductDto
        {
            Id = product.Id,
            Name = product.Name,
            Description = product.Description,
            Price = product.Price,
            StockQuantity = product.StockQuantity,
            CategoryName = product.Category?.Name ?? string.Empty,
            CreatedAt = product.CreatedAt
        };
    }

    private static IQueryable<Product> ApplySorting(
        IQueryable<Product> query, string? sortBy, string sortOrder)
    {
        var descending = sortOrder.Equals("desc", StringComparison.OrdinalIgnoreCase);

        return sortBy?.ToLower() switch
        {
            "name"     => descending ? query.OrderByDescending(p => p.Name)
                                     : query.OrderBy(p => p.Name),
            "price"    => descending ? query.OrderByDescending(p => p.Price)
                                     : query.OrderBy(p => p.Price),
            "stock"    => descending ? query.OrderByDescending(p => p.StockQuantity)
                                     : query.OrderBy(p => p.StockQuantity),
            "date"     => descending ? query.OrderByDescending(p => p.CreatedAt)
                                     : query.OrderBy(p => p.CreatedAt),
            "category" => descending ? query.OrderByDescending(p => p.Category.Name)
                                     : query.OrderBy(p => p.Category.Name),
            _          => query.OrderBy(p => p.Id)
        };
    }
}
```

### Example API Calls

List products with filtering, sorting, and pagination:

```http
GET /api/products?search=wireless&categoryId=3&minPrice=20&maxPrice=100&sortBy=price&sortOrder=asc&page=1&pageSize=10
```

Get a single product:

```http
GET /api/products/42
```

Create a product:

```http
POST /api/products
Content-Type: application/json

{
    "name": "Mechanical Keyboard",
    "description": "RGB mechanical keyboard with Cherry MX switches",
    "price": 129.99,
    "stockQuantity": 50,
    "categoryId": 3
}
```

Response:

```http
HTTP/1.1 201 Created
Location: /api/products/42
Content-Type: application/json

{
    "id": 42,
    "name": "Mechanical Keyboard",
    "description": "RGB mechanical keyboard with Cherry MX switches",
    "price": 129.99,
    "stockQuantity": 50,
    "inStock": true,
    "categoryName": "Peripherals",
    "createdAt": "2026-06-18T14:30:00Z"
}
```

Update a product:

```http
PUT /api/products/42
Content-Type: application/json

{
    "name": "Mechanical Keyboard Pro",
    "description": "Updated RGB mechanical keyboard with Cherry MX Brown switches",
    "price": 149.99,
    "stockQuantity": 45,
    "categoryId": 3
}
```

Delete a product:

```http
DELETE /api/products/42
```

> [!example]
> **Testing with curl:**
> ```bash
> # List products
> curl -s https://localhost:5001/api/products?page=1&pageSize=5 | jq
>
> # Create a product
> curl -s -X POST https://localhost:5001/api/products \
>   -H "Content-Type: application/json" \
>   -d '{"name":"Monitor","description":"27 inch 4K","price":499.99,"stockQuantity":20,"categoryId":1}' | jq
>
> # Update a product
> curl -s -X PUT https://localhost:5001/api/products/1 \
>   -H "Content-Type: application/json" \
>   -d '{"name":"Monitor Pro","description":"27 inch 4K HDR","price":599.99,"stockQuantity":15,"categoryId":1}'
>
> # Delete a product
> curl -s -X DELETE https://localhost:5001/api/products/1
> ```

> [!summary] Section Summary
> A production-quality API controller combines `[ApiController]`, `ControllerBase`, attribute routing, `ActionResult<T>` return types, separate DTOs for input and output, pagination, filtering, sorting, proper status codes, logging, and OpenAPI annotations. The complete `ProductsController` above demonstrates all of these patterns working together.

---

## Summary

> [!summary] Comprehensive Summary
> **API controllers** in ASP.NET Core are the foundation for building RESTful web services. The key concepts covered in this note:
>
> 1. **API controllers return data**, not views. They inherit from `ControllerBase` and are decorated with `[ApiController]`.
>
> 2. **`[ApiController]`** enables automatic model validation (400 responses), binding source inference, and ProblemDetails error formatting.
>
> 3. **`ControllerBase`** (not `Controller`) provides all necessary HTTP response helpers without view-rendering overhead.
>
> 4. **Attribute routing** with `[Route("api/[controller]")]` and HTTP verb attributes (`[HttpGet]`, `[HttpPost]`, etc.) defines the API surface.
>
> 5. **RESTful design** maps HTTP verbs to CRUD: GET for reading, POST for creating, PUT for full updates, PATCH for partial updates, DELETE for removal.
>
> 6. **`ActionResult<T>`** is the recommended return type -- it enables both flexible status code returns and automatic Swagger documentation.
>
> 7. **DTOs** decouple your API contract from domain models, preventing over-posting, hiding sensitive data, and enabling independent API evolution.
>
> 8. **Pagination** is mandatory for collection endpoints. Use `page`/`pageSize` parameters and return metadata (total count, total pages).
>
> 9. **Filtering and sorting** use query parameters with whitelist-based sorting to prevent injection attacks.
>
> 10. **API versioning** (preferably URL path: `/api/v1/`) allows breaking changes without breaking existing clients.
>
> These patterns combine into production-quality controllers like the `ProductsController` example, which demonstrates CRUD operations, DTO mapping, conditional filtering, safe sorting, pagination with metadata, and proper OpenAPI documentation.
>
> Related topics: [[Minimal APIs]] | [[Content Negotiation]] | [[API Conventions]]
