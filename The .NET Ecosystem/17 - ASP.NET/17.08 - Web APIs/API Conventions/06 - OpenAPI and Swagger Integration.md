---
tags:
  - csharp
  - asp-net-core
  - web-api
  - conventions
  - openapi
---


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
