---
tags:
  - csharp
  - asp-net-core
  - web-api
  - controllers
---


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
