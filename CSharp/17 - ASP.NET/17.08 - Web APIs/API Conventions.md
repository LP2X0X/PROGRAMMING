---
tags: [csharp, asp-net-core, web-api, conventions, openapi]
date: 2026-06-18
---

# API Conventions

Standardized patterns and practices for building consistent, well-documented, and predictable Web APIs in ASP.NET Core.

---

## Table of Contents

- [What Are API Conventions](#what-are-api-conventions)
- [ProducesResponseType Attribute](#producesresponsetype-attribute)
- [ApiConventionMethod and ApiConventionType](#apiconventionmethod-and-apiconventiontype)
- [Built-in DefaultApiConventions](#built-in-defaultapiconventions)
- [ProblemDetails (RFC 7807)](#problemdetails-rfc-7807)
- [OpenAPI and Swagger Integration](#openapi-and-swagger-integration)
- [API Versioning](#api-versioning)
- [HATEOAS](#hateoas)
- [Real-World Fully Documented API](#real-world-fully-documented-api)
- [Comprehensive Summary](#comprehensive-summary)

---

## What Are API Conventions

**API conventions** are ==standardized patterns that ensure consistent behavior, predictable responses, and clear documentation== across all endpoints in a Web API. Rather than leaving each developer to decide individually how errors are returned, what status codes map to what scenarios, or how API documentation is generated, conventions establish a uniform contract that both producers and consumers of the API can rely on.

API conventions in ASP.NET Core address several concerns:

- **Response type documentation** -- what status codes and response bodies each action can return
- **Error response format** -- a standardized structure for error details (ProblemDetails)
- **API documentation** -- auto-generated OpenAPI/Swagger specs from code metadata
- **Versioning** -- how breaking changes are introduced without disrupting existing consumers
- **Discoverability** -- making APIs self-describing via hypermedia links (HATEOAS)

Without conventions, APIs become inconsistent: one endpoint returns `{ "error": "Not found" }`, another returns `{ "message": "Resource missing", "code": 404 }`, and a third returns a bare string. Conventions eliminate this chaos.

> [!ad-note]
> Conventions are not just about documentation. They influence middleware behavior, exception handling, content negotiation, and how tools like Swagger UI render your API. See [[Content Negotiation]] for how response formatting interacts with these conventions.

> [!summary] Section Summary
> API conventions establish uniform patterns for response types, error formats, documentation, versioning, and discoverability across all endpoints in a Web API.

---

## ProducesResponseType Attribute

The **`[ProducesResponseType]`** attribute decorates controller actions to declare which HTTP status codes and response body types an action can return. This metadata is consumed by OpenAPI/Swagger tooling to generate accurate API documentation and by analyzers to warn about undocumented responses.

### Basic Syntax

```csharp
[HttpGet("{id}")]
[ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> GetProduct(int id)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null)
        return NotFound();

    return Ok(_mapper.Map<ProductDto>(product));
}
```

### Generic Syntax (.NET 7+)

Starting with .NET 7, a cleaner generic syntax is available:

```csharp
[HttpGet("{id}")]
[ProducesResponseType<ProductDto>(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> GetProduct(int id)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null)
        return NotFound();

    return Ok(_mapper.Map<ProductDto>(product));
}
```

### Common Patterns for CRUD Operations

```csharp
// GET collection
[HttpGet]
[ProducesResponseType<IEnumerable<ProductDto>>(StatusCodes.Status200OK)]
public async Task<IActionResult> GetAll() { /* ... */ }

// GET single
[HttpGet("{id}")]
[ProducesResponseType<ProductDto>(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> GetById(int id) { /* ... */ }

// POST create
[HttpPost]
[ProducesResponseType<ProductDto>(StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public async Task<IActionResult> Create(CreateProductDto dto) { /* ... */ }

// PUT update
[HttpPut("{id}")]
[ProducesResponseType(StatusCodes.Status204NoContent)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> Update(int id, UpdateProductDto dto) { /* ... */ }

// DELETE
[HttpDelete("{id}")]
[ProducesResponseType(StatusCodes.Status204NoContent)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> Delete(int id) { /* ... */ }
```

### Typed Return Values Reduce Boilerplate

When an action returns `ActionResult<T>`, the 200 response type is inferred automatically:

```csharp
[HttpGet("{id}")]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<ActionResult<ProductDto>> GetProduct(int id)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null)
        return NotFound();

    // 200 + ProductDto is inferred from ActionResult<ProductDto>
    return _mapper.Map<ProductDto>(product);
}
```

> [!tip]
> When using `ActionResult<T>`, the `[ProducesResponseType]` for the 200 status code with the `T` type is automatically inferred. You only need to declare non-success response types explicitly.

### Specifying Content Types

You can also declare the content type of the response:

```csharp
[HttpGet("{id}")]
[Produces("application/json")]
[ProducesResponseType<ProductDto>(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> GetProduct(int id) { /* ... */ }
```

The `[Produces]` attribute constrains the response content type and is separate from `[ProducesResponseType]`, which declares the status code and body type.

> [!warning]
> If you declare `[ProducesResponseType]` but your action can actually return status codes you did not declare, Swagger docs will be incomplete and consumers may not handle those cases. The `API1000` analyzer warns about this.

> [!summary] Section Summary
> `[ProducesResponseType]` declares possible HTTP status codes and response body types for each action, feeding Swagger documentation. The generic syntax `[ProducesResponseType<T>(...)]` is preferred in .NET 7+. Using `ActionResult<T>` auto-infers the success response type.

---

## ApiConventionMethod and ApiConventionType

Manually annotating every action with `[ProducesResponseType]` is repetitive. **`[ApiConventionMethod]`** and **`[ApiConventionType]`** let you apply response type conventions in bulk by matching action signatures to convention methods.

### ApiConventionType -- Apply to a Controller

```csharp
[ApiController]
[Route("api/[controller]")]
[ApiConventionType(typeof(DefaultApiConventions))]
public class ProductsController : ControllerBase
{
    // All actions in this controller inherit response type
    // metadata from DefaultApiConventions based on method
    // name and parameter matching.

    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> Get(int id) { /* ... */ }

    [HttpPost]
    public async Task<ActionResult<ProductDto>> Post(CreateProductDto dto) { /* ... */ }

    [HttpPut("{id}")]
    public async Task<IActionResult> Put(int id, UpdateProductDto dto) { /* ... */ }

    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id) { /* ... */ }
}
```

### ApiConventionMethod -- Apply to a Single Action

When you only want to apply conventions to a specific action, or when the default name matching does not apply:

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpPut("{id}/cancel")]
    [ApiConventionMethod(typeof(DefaultApiConventions), nameof(DefaultApiConventions.Put))]
    public async Task<IActionResult> CancelOrder(int id)
    {
        // This action doesn't follow the "Put" naming convention,
        // but we tell ASP.NET to treat it like a Put for conventions.
        var order = await _repository.GetByIdAsync(id);
        if (order is null) return NotFound();

        order.Status = OrderStatus.Cancelled;
        await _repository.UpdateAsync(order);
        return NoContent();
    }
}
```

### Assembly-Level Convention

You can apply conventions to every controller in the assembly:

```csharp
// In Program.cs or a dedicated AssemblyInfo.cs
[assembly: ApiConventionType(typeof(DefaultApiConventions))]
```

> [!ad-note]
> The convention matching is based on method names and parameter names. A method named `Get` with an `id` parameter matches `DefaultApiConventions.Get(int id)`. If your naming deviates, use `[ApiConventionMethod]` explicitly.

### Creating Custom Conventions

You can define your own convention type with custom response type metadata:

```csharp
public static class CustomApiConventions
{
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesDefaultResponseType]
    public static void Get(
        [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Suffix)]
        int id)
    { }

    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesDefaultResponseType]
    public static void Create(
        [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Any)]
        object model)
    { }

    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesDefaultResponseType]
    public static void Update(
        [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Suffix)]
        int id,
        [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Any)]
        object model)
    { }

    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesDefaultResponseType]
    public static void Delete(
        [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Suffix)]
        int id)
    { }
}
```

The `ApiConventionNameMatchBehavior` enum controls how parameter names are matched:

| Behavior | Description |
|---|---|
| `Any` | Matches any parameter name |
| `Exact` | Must match exactly |
| `Prefix` | Must start with the convention parameter name |
| `Suffix` | Must end with the convention parameter name |

Apply your custom conventions:

```csharp
[ApiConventionType(typeof(CustomApiConventions))]
public class ProductsController : ControllerBase { /* ... */ }
```

> [!summary] Section Summary
> `[ApiConventionType]` applies response type conventions to an entire controller or assembly. `[ApiConventionMethod]` targets individual actions. Custom conventions let you define project-specific response patterns with flexible name matching.

---

## Built-in DefaultApiConventions

ASP.NET Core ships with **`DefaultApiConventions`**, a static class that defines standard response type conventions for typical CRUD operations. Understanding what it covers helps you know when you can rely on it and when you need custom conventions.

### What DefaultApiConventions Defines

| Convention Method | Matched Action Names | Produces |
|---|---|---|
| `Get(id)` | `Get`, `Find`, `GetById` | 200, 404, default |
| `Post(model)` | `Post`, `Create` | 201, 400, default |
| `Put(id, model)` | `Put`, `Edit`, `Update` | 204, 400, 404, default |
| `Delete(id)` | `Delete`, `Remove` | 200, 400, 404, default |

### How Method Matching Works

`DefaultApiConventions` uses `[ApiConventionNameMatch]` on its methods:

1. **Method name** -- matched using `Prefix` behavior. An action named `GetProduct` matches `Get` because `GetProduct` starts with `Get`.
2. **Parameters** -- matched using `Any` or `Suffix` behavior depending on the convention method.

### Example: DefaultApiConventions in Action

```csharp
[assembly: ApiConventionType(typeof(DefaultApiConventions))]

namespace MyApi.Controllers;

[ApiController]
[Route("api/[controller]")]
public class CategoriesController : ControllerBase
{
    // Matches DefaultApiConventions.Get -- produces 200, 404
    [HttpGet("{id}")]
    public async Task<ActionResult<CategoryDto>> GetCategory(int id) { /* ... */ }

    // Matches DefaultApiConventions.Post -- produces 201, 400
    [HttpPost]
    public async Task<ActionResult<CategoryDto>> CreateCategory(
        CreateCategoryDto dto) { /* ... */ }

    // Matches DefaultApiConventions.Put -- produces 204, 400, 404
    [HttpPut("{id}")]
    public async Task<IActionResult> EditCategory(
        int id, UpdateCategoryDto dto) { /* ... */ }

    // Matches DefaultApiConventions.Delete -- produces 200, 400, 404
    [HttpDelete("{id}")]
    public async Task<IActionResult> RemoveCategory(int id) { /* ... */ }
}
```

> [!warning]
> `DefaultApiConventions` does not cover search/filter endpoints, batch operations, or non-CRUD actions like `Activate`, `Archive`, or `Export`. For those, you must use `[ProducesResponseType]` explicitly or create custom conventions.

> [!summary] Section Summary
> `DefaultApiConventions` provides out-of-the-box response type metadata for standard CRUD operations (Get, Post, Put, Delete). It matches by method name prefix and parameter patterns. Non-standard actions require explicit annotations or custom conventions.

---

## ProblemDetails (RFC 7807)

**ProblemDetails** is a ==standardized machine-readable format for describing errors in HTTP APIs==, defined in RFC 7807 (updated by RFC 9457). ASP.NET Core has built-in support for generating ProblemDetails responses, ensuring that every error your API returns follows a consistent, predictable structure.

### The ProblemDetails Structure

A ProblemDetails response is a JSON object with these properties:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
  "title": "Not Found",
  "status": 404,
  "detail": "Product with ID 42 was not found.",
  "instance": "/api/products/42",
  "traceId": "00-abc123def456-789ghi-00"
}
```

| Property | Description |
|---|---|
| `type` | A URI reference identifying the problem type. Defaults to the RFC section for the status code |
| `title` | A short, human-readable summary of the problem |
| `status` | The HTTP status code |
| `detail` | A human-readable explanation specific to this occurrence |
| `instance` | A URI reference identifying the specific occurrence (usually the request path) |
| `extensions` | Additional properties (like `traceId`) added by middleware or custom code |

### Enabling ProblemDetails Globally

In .NET 7+, you can enable ProblemDetails for all error responses with a single call:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddProblemDetails(); // Enable ProblemDetails globally

var app = builder.Build();

app.UseExceptionHandler();  // Converts unhandled exceptions to ProblemDetails
app.UseStatusCodePages();   // Converts empty error responses to ProblemDetails

app.MapControllers();
app.Run();
```

With this configuration:
- **Unhandled exceptions** return a 500 ProblemDetails response (without leaking stack traces in production)
- **Empty error responses** (like returning `NotFound()` with no body) are enriched with ProblemDetails
- **Model validation failures** already return ProblemDetails by default when using `[ApiController]`

### Validation Errors as ProblemDetails

When `[ApiController]` is applied, model validation failures automatically produce a **ValidationProblemDetails** response:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": ["The Name field is required."],
    "Price": ["The Price must be between 0.01 and 99999.99."]
  },
  "traceId": "00-abc123..."
}
```

### Customizing ProblemDetails Responses

#### Using AddProblemDetails with a Configure Action

```csharp
builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = context =>
    {
        // Add the trace ID to every ProblemDetails response
        context.ProblemDetails.Instance =
            $"{context.HttpContext.Request.Method} " +
            $"{context.HttpContext.Request.Path}";

        context.ProblemDetails.Extensions["requestId"] =
            context.HttpContext.TraceIdentifier;

        context.ProblemDetails.Extensions["nodeId"] =
            Environment.MachineName;
    };
});
```

#### Creating Custom Problem Types

Define domain-specific error types for richer error responses:

```csharp
[HttpPost]
public async Task<IActionResult> CreateOrder(CreateOrderDto dto)
{
    if (!await _inventoryService.HasSufficientStock(dto.ProductId, dto.Quantity))
    {
        return Problem(
            type: "https://myapi.com/errors/insufficient-stock",
            title: "Insufficient Stock",
            detail: $"Product {dto.ProductId} only has " +
                    $"{await _inventoryService.GetStock(dto.ProductId)} units " +
                    $"available, but {dto.Quantity} were requested.",
            statusCode: StatusCodes.Status409Conflict
        );
    }

    var order = await _orderService.CreateAsync(dto);
    return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
}
```

This produces:

```json
{
  "type": "https://myapi.com/errors/insufficient-stock",
  "title": "Insufficient Stock",
  "status": 409,
  "detail": "Product 7 only has 3 units available, but 10 were requested."
}
```

#### Custom Exception Handler with ProblemDetails

```csharp
app.UseExceptionHandler(exceptionApp =>
{
    exceptionApp.Run(async context =>
    {
        var exceptionFeature = context.Features
            .Get<IExceptionHandlerPathFeature>();
        var exception = exceptionFeature?.Error;

        var problemDetails = exception switch
        {
            EntityNotFoundException e => new ProblemDetails
            {
                Type = "https://myapi.com/errors/not-found",
                Title = "Resource Not Found",
                Status = StatusCodes.Status404NotFound,
                Detail = e.Message,
                Instance = context.Request.Path
            },
            BusinessRuleViolationException e => new ProblemDetails
            {
                Type = "https://myapi.com/errors/business-rule-violation",
                Title = "Business Rule Violation",
                Status = StatusCodes.Status422UnprocessableEntity,
                Detail = e.Message,
                Instance = context.Request.Path
            },
            _ => new ProblemDetails
            {
                Type = "https://tools.ietf.org/html/rfc9110#section-15.6.1",
                Title = "Internal Server Error",
                Status = StatusCodes.Status500InternalServerError,
                Detail = "An unexpected error occurred.",
                Instance = context.Request.Path
            }
        };

        context.Response.StatusCode = problemDetails.Status
                                      ?? StatusCodes.Status500InternalServerError;
        await context.Response.WriteAsJsonAsync(problemDetails);
    });
});
```

### ProblemDetails in Minimal APIs

ProblemDetails works the same way in [[Minimal APIs]]:

```csharp
app.MapGet("/api/products/{id}", async (int id, ProductRepository repo) =>
{
    var product = await repo.GetByIdAsync(id);
    return product is not null
        ? Results.Ok(product)
        : Results.Problem(
            title: "Product Not Found",
            detail: $"No product exists with ID {id}.",
            statusCode: StatusCodes.Status404NotFound);
});
```

> [!tip]
> In .NET 8+, `Results.Problem()` and `TypedResults.Problem()` support the full ProblemDetails parameter set. Use `TypedResults` for better OpenAPI metadata inference in [[Minimal APIs]].

> [!summary] Section Summary
> ProblemDetails (RFC 7807) provides a standardized JSON error format with `type`, `title`, `status`, `detail`, and `instance` fields. Enable globally with `AddProblemDetails()`, `UseExceptionHandler()`, and `UseStatusCodePages()`. Customize with domain-specific problem types and custom exception handlers.

---

## OpenAPI and Swagger Integration

**OpenAPI** (formerly Swagger) is the industry-standard specification for describing RESTful APIs. ASP.NET Core can auto-generate an OpenAPI document from your code and serve an interactive documentation UI. All the conventions discussed above (response types, ProblemDetails) feed directly into this generated documentation.

### .NET 9+ Built-in OpenAPI Support

Starting with .NET 9, ASP.NET Core includes built-in OpenAPI support via `Microsoft.AspNetCore.OpenApi`:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddOpenApi(); // Built-in OpenAPI document generation

var app = builder.Build();

app.MapOpenApi(); // Serves the OpenAPI document at /openapi/v1.json

app.MapControllers();
app.Run();
```

> [!ad-note]
> The built-in `AddOpenApi()` in .NET 9+ replaces the need for third-party packages like Swashbuckle or NSwag for basic OpenAPI document generation. However, for the Swagger UI, you still need Swashbuckle or a standalone UI like Scalar.

### Swashbuckle Setup (Pre-.NET 9 or for Swagger UI)

```bash
dotnet add package Swashbuckle.AspNetCore
```

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Product Catalog API",
        Version = "v1",
        Description = "API for managing product catalog, orders, and inventory.",
        Contact = new OpenApiContact
        {
            Name = "API Support",
            Email = "api-support@example.com"
        }
    });
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/swagger/v1/swagger.json", "Product Catalog v1");
        options.RoutePrefix = "swagger"; // Accessible at /swagger
    });
}

app.MapControllers();
app.Run();
```

Access the Swagger UI at `https://localhost:{port}/swagger`.

### Enriching Documentation with XML Comments

#### Step 1: Enable XML Documentation in the Project File

```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
  <NoWarn>$(NoWarn);1591</NoWarn> <!-- Suppress missing XML comment warnings -->
</PropertyGroup>
```

#### Step 2: Configure Swashbuckle to Use XML Comments

```csharp
builder.Services.AddSwaggerGen(options =>
{
    var xmlFilename = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    options.IncludeXmlComments(
        Path.Combine(AppContext.BaseDirectory, xmlFilename));
});
```

#### Step 3: Document Your Actions with XML Comments

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    /// <summary>
    /// Retrieves a specific product by its unique identifier.
    /// </summary>
    /// <param name="id">The unique identifier of the product.</param>
    /// <returns>The requested product.</returns>
    /// <response code="200">Returns the requested product.</response>
    /// <response code="404">If the product does not exist.</response>
    [HttpGet("{id}")]
    [ProducesResponseType<ProductDto>(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        var product = await _repository.GetByIdAsync(id);
        if (product is null)
            return NotFound();

        return _mapper.Map<ProductDto>(product);
    }

    /// <summary>
    /// Creates a new product in the catalog.
    /// </summary>
    /// <param name="dto">The product data.</param>
    /// <returns>The newly created product.</returns>
    /// <remarks>
    /// Sample request:
    ///
    ///     POST /api/products
    ///     {
    ///         "name": "Wireless Mouse",
    ///         "price": 29.99,
    ///         "categoryId": 3
    ///     }
    ///
    /// </remarks>
    /// <response code="201">Returns the newly created product.</response>
    /// <response code="400">If the product data is invalid.</response>
    [HttpPost]
    [ProducesResponseType<ProductDto>(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProductDto>> CreateProduct(
        CreateProductDto dto)
    {
        var product = await _service.CreateAsync(dto);
        return CreatedAtAction(
            nameof(GetProduct),
            new { id = product.Id },
            product);
    }
}
```

### Enriching Minimal API Documentation

For [[Minimal APIs]], use extension methods and `TypedResults`:

```csharp
app.MapGet("/api/products/{id}", async (int id, ProductRepository repo) =>
{
    var product = await repo.GetByIdAsync(id);
    return product is not null
        ? TypedResults.Ok(product)
        : TypedResults.NotFound();
})
.WithName("GetProduct")
.WithSummary("Retrieves a specific product by ID")
.WithDescription("Returns the full product details including category information.")
.WithTags("Products")
.Produces<ProductDto>(StatusCodes.Status200OK)
.Produces(StatusCodes.Status404NotFound);
```

### Using Scalar UI (.NET 9+)

.NET 9 projects often use Scalar instead of Swagger UI:

```bash
dotnet add package Scalar.AspNetCore
```

```csharp
app.MapScalarApiReference(); // Accessible at /scalar/v1
```

### Generating Client Code from OpenAPI Specs

Once you have an OpenAPI spec, you can generate strongly-typed client libraries.

#### Using NSwag

```bash
dotnet tool install -g NSwag.ConsoleCore
nswag openapi2csclient /input:swagger.json /output:ProductApiClient.cs /namespace:MyApp.ApiClients
```

#### Using Microsoft Kiota

```bash
dotnet tool install -g Microsoft.OpenApi.Kiota
kiota generate -l CSharp -d https://localhost:5001/openapi/v1.json -o ./ApiClient -n MyApp.ApiClient
```

#### Using Visual Studio Connected Services

Visual Studio can generate clients automatically:
1. Right-click the project and select **Add > Connected Service**
2. Choose **OpenAPI**
3. Provide the URL or file path to the OpenAPI document
4. A typed client is generated and added to the project

> [!example]
> The generated OpenAPI document at `/openapi/v1.json` or `/swagger/v1/swagger.json` is machine-readable. CI/CD pipelines can use it to:
> - Generate client SDKs for frontend teams
> - Run contract tests to detect breaking changes
> - Publish documentation to API portals

> [!summary] Section Summary
> OpenAPI/Swagger auto-generates interactive API documentation from code metadata. Use `AddOpenApi()` (.NET 9+) or Swashbuckle for document generation. Enrich docs with XML comments, `[ProducesResponseType]`, and Minimal API extension methods. Generate client code with NSwag, Kiota, or Visual Studio Connected Services.

---

## API Versioning

**API versioning** allows you to evolve your API without breaking existing consumers. ==When you need to introduce breaking changes, a new version lets old clients continue using the previous contract while new clients adopt the updated one.==

### Installing the Versioning Package

```bash
dotnet add package Asp.Versioning.Mvc
dotnet add package Asp.Versioning.Mvc.ApiExplorer
```

### Configuring API Versioning

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true; // Adds api-supported-versions header
    options.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),
        new QueryStringApiVersionReader("api-version"),
        new HeaderApiVersionReader("X-Api-Version"),
        new MediaTypeApiVersionReader("v")
    );
})
.AddApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});
```

### Versioning Strategies

#### URL Segment Versioning (Most Common)

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0")]
public class ProductsV1Controller : ControllerBase
{
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDtoV1>> GetProduct(int id)
    {
        // V1 response shape
        return new ProductDtoV1
        {
            Id = id,
            Name = "Wireless Mouse",
            Price = 29.99m
        };
    }
}

[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("2.0")]
public class ProductsV2Controller : ControllerBase
{
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDtoV2>> GetProduct(int id)
    {
        // V2 response includes additional fields
        return new ProductDtoV2
        {
            Id = id,
            Name = "Wireless Mouse",
            Price = new MoneyDto { Amount = 29.99m, Currency = "USD" },
            Sku = "WM-001",
            CreatedAt = DateTime.UtcNow
        };
    }
}
```

Requests:

```http
GET /api/v1/products/42 HTTP/1.1
Host: localhost:5001
```

```http
GET /api/v2/products/42 HTTP/1.1
Host: localhost:5001
```

#### Query String Versioning

```http
GET /api/products/42?api-version=2.0 HTTP/1.1
Host: localhost:5001
```

#### Header Versioning

```http
GET /api/products/42 HTTP/1.1
Host: localhost:5001
X-Api-Version: 2.0
```

### Deprecating API Versions

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0", Deprecated = true)] // Marks v1 as deprecated
[ApiVersion("2.0")]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    [MapToApiVersion("1.0")]
    public ActionResult<ProductDtoV1> GetProductV1(int id) { /* ... */ }

    [HttpGet("{id}")]
    [MapToApiVersion("2.0")]
    public ActionResult<ProductDtoV2> GetProductV2(int id) { /* ... */ }
}
```

Deprecated versions include an `api-deprecated-versions` response header, signaling to consumers that they should migrate.

### Swagger Integration with Versioning

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Product API",
        Version = "v1",
        Description = "Version 1.0 - Deprecated"
    });
    options.SwaggerDoc("v2", new OpenApiInfo
    {
        Title = "Product API",
        Version = "v2",
        Description = "Version 2.0 - Current"
    });
});

// In the middleware pipeline
app.UseSwaggerUI(options =>
{
    options.SwaggerEndpoint("/swagger/v1/swagger.json", "Product API v1 (Deprecated)");
    options.SwaggerEndpoint("/swagger/v2/swagger.json", "Product API v2");
});
```

> [!tip]
> URL segment versioning (`/api/v2/...`) is the most widely adopted strategy because it is the most visible, cacheable, and easy to route. Query string and header versioning can be useful for internal APIs or when URL changes are undesirable.

> [!summary] Section Summary
> API versioning uses `Asp.Versioning.Mvc` to support URL segment, query string, header, and media type strategies. URL segment versioning is the most common. Deprecated versions are signaled via response headers. Swagger can be configured to show separate docs per version.

---

## HATEOAS

**HATEOAS** (Hypermedia As The Engine Of Application State) is a REST constraint where ==API responses include hyperlinks that tell the client what actions are available next==. Instead of clients hardcoding URL patterns, the server guides navigation dynamically.

### Basic HATEOAS Response

```csharp
public class ProductResource
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public List<LinkDto> Links { get; set; } = [];
}

public class LinkDto
{
    public string Href { get; set; } = string.Empty;
    public string Rel { get; set; } = string.Empty;  // Relation type
    public string Method { get; set; } = string.Empty;
}
```

### Generating Links in a Controller

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<ProductResource>> GetProduct(int id)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null) return NotFound();

    var resource = _mapper.Map<ProductResource>(product);
    resource.Links = new List<LinkDto>
    {
        new LinkDto
        {
            Href = Url.Action(nameof(GetProduct), new { id })!,
            Rel = "self",
            Method = "GET"
        },
        new LinkDto
        {
            Href = Url.Action(nameof(UpdateProduct), new { id })!,
            Rel = "update",
            Method = "PUT"
        },
        new LinkDto
        {
            Href = Url.Action(nameof(DeleteProduct), new { id })!,
            Rel = "delete",
            Method = "DELETE"
        }
    };

    return resource;
}
```

Response:

```json
{
  "id": 42,
  "name": "Wireless Mouse",
  "price": 29.99,
  "links": [
    { "href": "/api/products/42", "rel": "self", "method": "GET" },
    { "href": "/api/products/42", "rel": "update", "method": "PUT" },
    { "href": "/api/products/42", "rel": "delete", "method": "DELETE" }
  ]
}
```

> [!ad-note]
> HATEOAS is a full REST maturity level (Richardson Maturity Model Level 3) but is rarely implemented in practice. Most real-world APIs stop at Level 2 (HTTP verbs + resource URIs). Consider HATEOAS for public APIs with many consumers who need discoverability.

> [!summary] Section Summary
> HATEOAS adds hyperlinks to API responses that describe available actions and navigation. While a core REST principle, it is uncommon in practice. Most APIs rely on versioned documentation instead of hypermedia-driven discovery.

---

## Real-World Fully Documented API

This section puts everything together: a production-style API with ProblemDetails, Swagger, versioning, response type annotations, and XML documentation.

### Project Configuration

```xml
<!-- ProductCatalog.Api.csproj -->
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <NoWarn>$(NoWarn);1591</NoWarn>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Asp.Versioning.Mvc" Version="8.1.0" />
    <PackageReference Include="Asp.Versioning.Mvc.ApiExplorer" Version="8.1.0" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="7.2.0" />
  </ItemGroup>
</Project>
```

### Program.cs -- Full Service Configuration

```csharp
using System.Reflection;
using Asp.Versioning;
using Microsoft.OpenApi.Models;

var builder = WebApplication.CreateBuilder(args);

// --- Controllers ---
builder.Services.AddControllers();

// --- ProblemDetails ---
builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = context =>
    {
        context.ProblemDetails.Extensions["traceId"] =
            context.HttpContext.TraceIdentifier;
        context.ProblemDetails.Instance =
            context.HttpContext.Request.Path;
    };
});

// --- API Versioning ---
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
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

// --- Swagger / OpenAPI ---
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Product Catalog API",
        Version = "v1",
        Description = "Version 1 -- Basic product management. **Deprecated.**"
    });
    options.SwaggerDoc("v2", new OpenApiInfo
    {
        Title = "Product Catalog API",
        Version = "v2",
        Description = "Version 2 -- Extended product data with SKU and pricing."
    });

    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    options.IncludeXmlComments(
        Path.Combine(AppContext.BaseDirectory, xmlFile));
});

var app = builder.Build();

// --- Middleware Pipeline ---
app.UseExceptionHandler();
app.UseStatusCodePages();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/swagger/v1/swagger.json",
            "Product Catalog v1 (Deprecated)");
        options.SwaggerEndpoint("/swagger/v2/swagger.json",
            "Product Catalog v2");
    });
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### DTOs

```csharp
namespace ProductCatalog.Api.Dtos;

public record ProductDtoV1(
    int Id,
    string Name,
    decimal Price,
    string Category);

public record ProductDtoV2(
    int Id,
    string Name,
    MoneyDto Price,
    string Sku,
    string Category,
    DateTime CreatedAtUtc,
    DateTime? UpdatedAtUtc);

public record MoneyDto(decimal Amount, string Currency);

public record CreateProductDto(
    string Name,
    decimal Price,
    string Currency,
    int CategoryId);

public record UpdateProductDto(
    string Name,
    decimal Price,
    string Currency);
```

### V1 Controller (Deprecated)

```csharp
namespace ProductCatalog.Api.Controllers.V1;

/// <summary>
/// Manages products in the catalog (V1 -- Deprecated).
/// </summary>
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0", Deprecated = true)]
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    private readonly IProductRepository _repository;

    public ProductsController(IProductRepository repository)
    {
        _repository = repository;
    }

    /// <summary>
    /// Retrieves all products.
    /// </summary>
    /// <returns>A list of all products.</returns>
    [HttpGet]
    [ProducesResponseType<IEnumerable<ProductDtoV1>>(StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<ProductDtoV1>>> GetAll()
    {
        var products = await _repository.GetAllAsync();
        return Ok(products.Select(p => new ProductDtoV1(
            p.Id, p.Name, p.Price, p.Category.Name)));
    }

    /// <summary>
    /// Retrieves a specific product by ID.
    /// </summary>
    /// <param name="id">The product identifier.</param>
    /// <returns>The product if found.</returns>
    /// <response code="200">Returns the product.</response>
    /// <response code="404">Product not found.</response>
    [HttpGet("{id}")]
    [ProducesResponseType<ProductDtoV1>(StatusCodes.Status200OK)]
    [ProducesResponseType<ProblemDetails>(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductDtoV1>> Get(int id)
    {
        var product = await _repository.GetByIdAsync(id);
        if (product is null)
        {
            return Problem(
                title: "Product Not Found",
                detail: $"No product exists with ID {id}.",
                statusCode: StatusCodes.Status404NotFound);
        }

        return new ProductDtoV1(
            product.Id, product.Name, product.Price, product.Category.Name);
    }
}
```

### V2 Controller (Current)

```csharp
namespace ProductCatalog.Api.Controllers.V2;

/// <summary>
/// Manages products in the catalog (V2).
/// </summary>
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("2.0")]
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    private readonly IProductRepository _repository;
    private readonly IProductService _service;

    public ProductsController(
        IProductRepository repository,
        IProductService service)
    {
        _repository = repository;
        _service = service;
    }

    /// <summary>
    /// Retrieves all products with extended details.
    /// </summary>
    /// <returns>A list of all products with SKU, pricing, and timestamps.</returns>
    [HttpGet]
    [ProducesResponseType<IEnumerable<ProductDtoV2>>(StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<ProductDtoV2>>> GetAll()
    {
        var products = await _repository.GetAllAsync();
        return Ok(products.Select(MapToV2Dto));
    }

    /// <summary>
    /// Retrieves a specific product by ID with extended details.
    /// </summary>
    /// <param name="id">The product identifier.</param>
    /// <returns>The product with full details.</returns>
    /// <response code="200">Returns the product with SKU and pricing.</response>
    /// <response code="404">Product not found -- returns ProblemDetails.</response>
    [HttpGet("{id}")]
    [ProducesResponseType<ProductDtoV2>(StatusCodes.Status200OK)]
    [ProducesResponseType<ProblemDetails>(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductDtoV2>> Get(int id)
    {
        var product = await _repository.GetByIdAsync(id);
        if (product is null)
        {
            return Problem(
                title: "Product Not Found",
                detail: $"No product exists with ID {id}.",
                statusCode: StatusCodes.Status404NotFound);
        }

        return MapToV2Dto(product);
    }

    /// <summary>
    /// Creates a new product in the catalog.
    /// </summary>
    /// <param name="dto">The product creation data.</param>
    /// <returns>The newly created product with generated SKU.</returns>
    /// <remarks>
    /// Sample request:
    ///
    ///     POST /api/v2/products
    ///     {
    ///         "name": "Wireless Mouse",
    ///         "price": 29.99,
    ///         "currency": "USD",
    ///         "categoryId": 3
    ///     }
    ///
    /// </remarks>
    /// <response code="201">Product created successfully.</response>
    /// <response code="400">Invalid product data -- returns ValidationProblemDetails.</response>
    /// <response code="409">Duplicate product name -- returns ProblemDetails.</response>
    [HttpPost]
    [ProducesResponseType<ProductDtoV2>(StatusCodes.Status201Created)]
    [ProducesResponseType<ValidationProblemDetails>(StatusCodes.Status400BadRequest)]
    [ProducesResponseType<ProblemDetails>(StatusCodes.Status409Conflict)]
    public async Task<ActionResult<ProductDtoV2>> Create(CreateProductDto dto)
    {
        if (await _repository.ExistsByNameAsync(dto.Name))
        {
            return Problem(
                type: "https://myapi.com/errors/duplicate-product",
                title: "Duplicate Product",
                detail: $"A product named '{dto.Name}' already exists.",
                statusCode: StatusCodes.Status409Conflict);
        }

        var product = await _service.CreateAsync(dto);
        var result = MapToV2Dto(product);

        return CreatedAtAction(
            nameof(Get),
            new { id = product.Id },
            result);
    }

    /// <summary>
    /// Updates an existing product.
    /// </summary>
    /// <param name="id">The product identifier.</param>
    /// <param name="dto">The updated product data.</param>
    /// <response code="204">Product updated successfully.</response>
    /// <response code="400">Invalid product data.</response>
    /// <response code="404">Product not found.</response>
    [HttpPut("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType<ValidationProblemDetails>(StatusCodes.Status400BadRequest)]
    [ProducesResponseType<ProblemDetails>(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> Update(int id, UpdateProductDto dto)
    {
        var product = await _repository.GetByIdAsync(id);
        if (product is null)
        {
            return Problem(
                title: "Product Not Found",
                detail: $"No product exists with ID {id}.",
                statusCode: StatusCodes.Status404NotFound);
        }

        await _service.UpdateAsync(id, dto);
        return NoContent();
    }

    /// <summary>
    /// Deletes a product from the catalog.
    /// </summary>
    /// <param name="id">The product identifier.</param>
    /// <response code="204">Product deleted successfully.</response>
    /// <response code="404">Product not found.</response>
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType<ProblemDetails>(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> Delete(int id)
    {
        var product = await _repository.GetByIdAsync(id);
        if (product is null)
        {
            return Problem(
                title: "Product Not Found",
                detail: $"No product exists with ID {id}.",
                statusCode: StatusCodes.Status404NotFound);
        }

        await _repository.DeleteAsync(id);
        return NoContent();
    }

    private static ProductDtoV2 MapToV2Dto(Product product)
    {
        return new ProductDtoV2(
            product.Id,
            product.Name,
            new MoneyDto(product.Price, product.Currency),
            product.Sku,
            product.Category.Name,
            product.CreatedAtUtc,
            product.UpdatedAtUtc);
    }
}
```

### Testing the API

```http
### Get all products (v2)
GET https://localhost:5001/api/v2/products HTTP/1.1
Accept: application/json

### Get single product (v2)
GET https://localhost:5001/api/v2/products/42 HTTP/1.1
Accept: application/json

### Create a product (v2)
POST https://localhost:5001/api/v2/products HTTP/1.1
Content-Type: application/json

{
  "name": "Wireless Mouse",
  "price": 29.99,
  "currency": "USD",
  "categoryId": 3
}

### Update a product (v2)
PUT https://localhost:5001/api/v2/products/42 HTTP/1.1
Content-Type: application/json

{
  "name": "Wireless Mouse Pro",
  "price": 39.99,
  "currency": "USD"
}

### Delete a product (v2)
DELETE https://localhost:5001/api/v2/products/42 HTTP/1.1

### Get product from deprecated v1
GET https://localhost:5001/api/v1/products/42 HTTP/1.1
Accept: application/json
```

### Sample Responses

Successful response (GET /api/v2/products/42):

```json
{
  "id": 42,
  "name": "Wireless Mouse",
  "price": {
    "amount": 29.99,
    "currency": "USD"
  },
  "sku": "WM-001",
  "category": "Peripherals",
  "createdAtUtc": "2026-03-15T10:30:00Z",
  "updatedAtUtc": null
}
```

Error response (GET /api/v2/products/9999):

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
  "title": "Product Not Found",
  "status": 404,
  "detail": "No product exists with ID 9999.",
  "instance": "/api/v2/products/9999",
  "traceId": "00-8a4f3b2c1d0e9f8a-7b6c5d4e3f2a1b0c-00"
}
```

Validation error response (POST with missing Name):

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": ["The Name field is required."],
    "CategoryId": ["The CategoryId must be greater than 0."]
  },
  "traceId": "00-1a2b3c4d5e6f7a8b-9c0d1e2f3a4b5c6d-00"
}
```

> [!summary] Section Summary
> A production-ready API combines ProblemDetails for consistent error responses, `[ProducesResponseType]` for accurate Swagger documentation, API versioning for backward compatibility, and XML comments for rich descriptions. Together these conventions create a self-documenting, predictable API contract.

---

## Comprehensive Summary

> [!summary] Comprehensive Summary
> **API conventions** in ASP.NET Core standardize how endpoints declare their behavior, handle errors, generate documentation, and evolve over time.
>
> **Key takeaways:**
>
> - **`[ProducesResponseType]`** declares possible response status codes and body types per action, feeding directly into OpenAPI/Swagger documentation. Use the generic form `[ProducesResponseType<T>(...)]` in .NET 7+.
>
> - **`[ApiConventionType]`** and **`[ApiConventionMethod]`** eliminate repetitive response type annotations by applying conventions in bulk based on method name and parameter matching. `DefaultApiConventions` covers standard CRUD patterns.
>
> - **ProblemDetails (RFC 7807)** provides a machine-readable, standardized error format. Enable globally with `AddProblemDetails()`, `UseExceptionHandler()`, and `UseStatusCodePages()`. Customize with domain-specific problem types for business rule violations.
>
> - **OpenAPI/Swagger** auto-generates interactive documentation. Use `AddOpenApi()` (.NET 9+) or Swashbuckle. Enrich with XML comments, `<summary>`, `<remarks>`, and `<response>` tags. Generate typed clients with NSwag or Kiota.
>
> - **API versioning** via `Asp.Versioning.Mvc` supports URL segment, query string, header, and media type strategies. URL segment versioning (`/api/v2/...`) is the most common. Mark old versions as deprecated.
>
> - **HATEOAS** adds hypermedia links to responses for discoverability, but is rarely implemented in practice. Most APIs rely on versioned documentation instead.
>
> These conventions work together: response type attributes feed Swagger, ProblemDetails standardize error responses across versions, and XML comments enrich the generated documentation. A well-conventioned API is self-documenting, predictable, and maintainable.
>
> See also: [[API Controllers]] for controller-level configuration, [[Minimal APIs]] for the alternative endpoint model, and [[Content Negotiation]] for response formatting.
