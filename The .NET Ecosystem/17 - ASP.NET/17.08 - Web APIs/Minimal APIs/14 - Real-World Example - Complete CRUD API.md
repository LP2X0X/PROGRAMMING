---
tags:
  - csharp
  - asp-net-core
  - minimal-apis
  - web-api
---


This section brings everything together into a production-style minimal API for managing products, with DI, validation, Swagger, and proper error handling.

### Project Structure

```
ProductApi/
  Program.cs
  Models/
    Product.cs
    CreateProductDto.cs
    UpdateProductDto.cs
    ProductResponse.cs
  Services/
    IProductService.cs
    ProductService.cs
  Validators/
    CreateProductValidator.cs
    UpdateProductValidator.cs
  Endpoints/
    ProductEndpoints.cs
  Data/
    AppDbContext.cs
```

### Models

```csharp
// Models/Product.cs
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Sku { get; set; } = string.Empty;
    public string? Description { get; set; }
    public decimal Price { get; set; }
    public int CategoryId { get; set; }
    public bool IsActive { get; set; } = true;
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
}

// Models/CreateProductDto.cs
public record CreateProductDto(
    string Name,
    string Sku,
    string? Description,
    decimal Price,
    int CategoryId);

// Models/UpdateProductDto.cs
public record UpdateProductDto(
    string Name,
    string? Description,
    decimal Price,
    int CategoryId,
    bool IsActive);

// Models/ProductResponse.cs
public record ProductResponse(
    int Id,
    string Name,
    string Sku,
    string? Description,
    decimal Price,
    int CategoryId,
    bool IsActive,
    DateTime CreatedAt);
```

### DbContext

```csharp
// Data/AppDbContext.cs
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products => Set<Product>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Product>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).HasMaxLength(200).IsRequired();
            entity.Property(e => e.Sku).HasMaxLength(20).IsRequired();
            entity.HasIndex(e => e.Sku).IsUnique();
            entity.Property(e => e.Price).HasPrecision(18, 2);
            entity.Property(e => e.Description).HasMaxLength(2000);
        });
    }
}
```

### Service Layer

```csharp
// Services/IProductService.cs
public interface IProductService
{
    Task<IEnumerable<ProductResponse>> GetAllAsync(
        string? name, int page, int pageSize);
    Task<ProductResponse?> GetByIdAsync(int id);
    Task<ProductResponse> CreateAsync(CreateProductDto dto);
    Task<ProductResponse?> UpdateAsync(int id, UpdateProductDto dto);
    Task<bool> DeleteAsync(int id);
    Task<bool> SkuExistsAsync(string sku, int? excludeId = null);
}

// Services/ProductService.cs
public class ProductService : IProductService
{
    private readonly AppDbContext _db;

    public ProductService(AppDbContext db) => _db = db;

    public async Task<IEnumerable<ProductResponse>> GetAllAsync(
        string? name, int page, int pageSize)
    {
        var query = _db.Products.AsQueryable();

        if (!string.IsNullOrWhiteSpace(name))
            query = query.Where(p => p.Name.Contains(name));

        return await query
            .OrderBy(p => p.Name)
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .Select(p => ToResponse(p))
            .ToListAsync();
    }

    public async Task<ProductResponse?> GetByIdAsync(int id)
    {
        var product = await _db.Products.FindAsync(id);
        return product is null ? null : ToResponse(product);
    }

    public async Task<ProductResponse> CreateAsync(CreateProductDto dto)
    {
        var product = new Product
        {
            Name = dto.Name,
            Sku = dto.Sku,
            Description = dto.Description,
            Price = dto.Price,
            CategoryId = dto.CategoryId,
            CreatedAt = DateTime.UtcNow
        };

        _db.Products.Add(product);
        await _db.SaveChangesAsync();

        return ToResponse(product);
    }

    public async Task<ProductResponse?> UpdateAsync(int id, UpdateProductDto dto)
    {
        var product = await _db.Products.FindAsync(id);
        if (product is null) return null;

        product.Name = dto.Name;
        product.Description = dto.Description;
        product.Price = dto.Price;
        product.CategoryId = dto.CategoryId;
        product.IsActive = dto.IsActive;
        product.UpdatedAt = DateTime.UtcNow;

        await _db.SaveChangesAsync();
        return ToResponse(product);
    }

    public async Task<bool> DeleteAsync(int id)
    {
        var product = await _db.Products.FindAsync(id);
        if (product is null) return false;

        _db.Products.Remove(product);
        await _db.SaveChangesAsync();
        return true;
    }

    public async Task<bool> SkuExistsAsync(string sku, int? excludeId = null)
    {
        return await _db.Products
            .AnyAsync(p => p.Sku == sku && (!excludeId.HasValue || p.Id != excludeId));
    }

    private static ProductResponse ToResponse(Product p) =>
        new(p.Id, p.Name, p.Sku, p.Description, p.Price,
            p.CategoryId, p.IsActive, p.CreatedAt);
}
```

### Validators (FluentValidation)

```csharp
// Validators/CreateProductValidator.cs
public class CreateProductValidator : AbstractValidator<CreateProductDto>
{
    private readonly IProductService _service;

    public CreateProductValidator(IProductService service)
    {
        _service = service;

        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Product name is required.")
            .MaximumLength(200).WithMessage("Name cannot exceed 200 characters.");

        RuleFor(x => x.Sku)
            .NotEmpty().WithMessage("SKU is required.")
            .Matches(@"^[A-Z]{2}-\d{4}$").WithMessage("SKU format: XX-0000")
            .MustAsync(async (sku, ct) => !await _service.SkuExistsAsync(sku))
            .WithMessage("SKU already exists.");

        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Price must be positive.")
            .LessThanOrEqualTo(999999.99m).WithMessage("Price cannot exceed 999,999.99.");

        RuleFor(x => x.CategoryId)
            .GreaterThan(0).WithMessage("Valid category required.");
    }
}

// Validators/UpdateProductValidator.cs
public class UpdateProductValidator : AbstractValidator<UpdateProductDto>
{
    public UpdateProductValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Product name is required.")
            .MaximumLength(200).WithMessage("Name cannot exceed 200 characters.");

        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Price must be positive.");

        RuleFor(x => x.CategoryId)
            .GreaterThan(0).WithMessage("Valid category required.");
    }
}
```

### Reusable Validation Filter

```csharp
// Filters/FluentValidationFilter.cs
public class FluentValidationFilter<T> : IEndpointFilter where T : class
{
    public async ValueTask<object?> InvokeAsync(
        EndpointFilterInvocationContext context,
        EndpointFilterDelegate next)
    {
        var dto = context.Arguments.OfType<T>().FirstOrDefault();

        if (dto is null)
            return Results.BadRequest(new { error = "Request body is required." });

        var validator = context.HttpContext.RequestServices
            .GetService<IValidator<T>>();

        if (validator is not null)
        {
            var result = await validator.ValidateAsync(dto);
            if (!result.IsValid)
            {
                return Results.ValidationProblem(
                    result.Errors
                        .GroupBy(e => e.PropertyName)
                        .ToDictionary(
                            g => g.Key,
                            g => g.Select(e => e.ErrorMessage).ToArray()));
            }
        }

        return await next(context);
    }
}
```

### Endpoint Registration

```csharp
// Endpoints/ProductEndpoints.cs
public static class ProductEndpoints
{
    public static void MapProductEndpoints(this WebApplication app)
    {
        var group = app.MapGroup("/api/products")
            .WithTags("Products")
            .WithOpenApi();

        group.MapGet("/", GetAll)
            .WithName("GetProducts")
            .WithSummary("Search products with pagination")
            .Produces<IEnumerable<ProductResponse>>(200);

        group.MapGet("/{id:int}", GetById)
            .WithName("GetProductById")
            .WithSummary("Get a product by ID")
            .Produces<ProductResponse>(200)
            .Produces(404);

        group.MapPost("/", Create)
            .WithName("CreateProduct")
            .WithSummary("Create a new product")
            .Produces<ProductResponse>(201)
            .ProducesValidationProblem()
            .AddEndpointFilter<FluentValidationFilter<CreateProductDto>>();

        group.MapPut("/{id:int}", Update)
            .WithName("UpdateProduct")
            .WithSummary("Update an existing product")
            .Produces<ProductResponse>(200)
            .Produces(404)
            .ProducesValidationProblem()
            .AddEndpointFilter<FluentValidationFilter<UpdateProductDto>>();

        group.MapDelete("/{id:int}", Delete)
            .WithName("DeleteProduct")
            .WithSummary("Delete a product")
            .Produces(204)
            .Produces(404);
    }

    private static async Task<IResult> GetAll(
        [FromQuery] string? name,
        [FromQuery] int? page,
        [FromQuery] int? pageSize,
        IProductService service)
    {
        var products = await service.GetAllAsync(
            name,
            page ?? 1,
            Math.Clamp(pageSize ?? 20, 1, 100));

        return Results.Ok(products);
    }

    private static async Task<IResult> GetById(
        int id, IProductService service)
    {
        var product = await service.GetByIdAsync(id);
        return product is not null
            ? Results.Ok(product)
            : Results.NotFound(new { error = $"Product {id} not found." });
    }

    private static async Task<IResult> Create(
        CreateProductDto dto, IProductService service)
    {
        var product = await service.CreateAsync(dto);
        return Results.Created($"/api/products/{product.Id}", product);
    }

    private static async Task<IResult> Update(
        int id, UpdateProductDto dto, IProductService service)
    {
        var product = await service.UpdateAsync(id, dto);
        return product is not null
            ? Results.Ok(product)
            : Results.NotFound(new { error = $"Product {id} not found." });
    }

    private static async Task<IResult> Delete(
        int id, IProductService service)
    {
        var deleted = await service.DeleteAsync(id);
        return deleted
            ? Results.NoContent()
            : Results.NotFound(new { error = $"Product {id} not found." });
    }
}
```

### Program.cs

```csharp
using FluentValidation;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Database
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));

// Services
builder.Services.AddScoped<IProductService, ProductService>();

// Validation
builder.Services.AddValidatorsFromAssemblyContaining<CreateProductValidator>();

// OpenAPI / Swagger
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new Microsoft.OpenApi.Models.OpenApiInfo
    {
        Title = "Product API",
        Version = "v1",
        Description = "A minimal API for managing products"
    });
});

var app = builder.Build();

// Middleware
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Map endpoints
app.MapProductEndpoints();

// Health check
app.MapGet("/health", () => Results.Ok(new
{
    status = "Healthy",
    timestamp = DateTime.UtcNow
}))
.WithTags("System")
.ExcludeFromDescription();

app.Run();
```

### Testing the API

```http
### Get all products
GET https://localhost:5001/api/products?page=1&pageSize=10

### Get product by ID
GET https://localhost:5001/api/products/1

### Create a product
POST https://localhost:5001/api/products
Content-Type: application/json

{
    "name": "Wireless Mouse",
    "sku": "WM-1001",
    "description": "Ergonomic wireless mouse with Bluetooth",
    "price": 29.99,
    "categoryId": 3
}

### Update a product
PUT https://localhost:5001/api/products/1
Content-Type: application/json

{
    "name": "Wireless Mouse Pro",
    "description": "Updated ergonomic wireless mouse",
    "price": 39.99,
    "categoryId": 3,
    "isActive": true
}

### Delete a product
DELETE https://localhost:5001/api/products/1
```

```json
// Example validation error response (RFC 7807)
{
    "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "errors": {
        "Name": ["Product name is required."],
        "Sku": ["SKU format: XX-0000"],
        "Price": ["Price must be positive."]
    }
}
```

> [!summary] Section Summary
> A production-grade minimal API uses route groups for organization, FluentValidation with endpoint filters for validation, a service layer for business logic, DTOs for request/response shaping, and Swagger for documentation. Handlers are organized into static classes with extension methods, keeping `Program.cs` clean.
