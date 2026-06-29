---
tags:
  - csharp
  - asp-net-core
  - web-api
  - conventions
  - openapi
---


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
