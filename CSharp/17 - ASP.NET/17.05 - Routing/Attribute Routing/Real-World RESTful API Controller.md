---
tags:
  - csharp
  - asp-net-core
  - routing
  - attribute-routing
---


Here is a complete, production-style API controller demonstrating the full range of attribute routing features:

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace MyApp.Controllers;

/// <summary>
/// Manages product resources.
/// Demonstrates attribute routing best practices for a RESTful API.
/// </summary>
[Route("api/[controller]")]          // Prefix: api/Products
[ApiController]                       // Requires attribute routing, enables auto-validation
[Produces("application/json")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _service;

    public ProductsController(IProductService service)
    {
        _service = service;
    }

    /// GET api/Products
    /// Returns all products with optional filtering.
    [HttpGet]
    public async Task<ActionResult<IEnumerable<ProductDto>>> GetAll(
        [FromQuery] string? category,
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 20)
    {
        var products = await _service.GetAllAsync(category, page, pageSize);
        return Ok(products);
    }

    /// GET api/Products/5
    /// Returns a single product by ID.
    [HttpGet("{id:int}", Name = "GetProduct")]
    public async Task<ActionResult<ProductDto>> GetById(int id)
    {
        var product = await _service.GetByIdAsync(id);
        if (product is null) return NotFound();
        return Ok(product);
    }

    /// GET api/Products/by-slug/wireless-mouse
    /// Returns a product by its URL slug.
    [HttpGet("by-slug/{slug:regex(^[[a-z0-9-]]+$)}")]
    public async Task<ActionResult<ProductDto>> GetBySlug(string slug)
    {
        var product = await _service.GetBySlugAsync(slug);
        if (product is null) return NotFound();
        return Ok(product);
    }

    /// GET api/Products/5/reviews
    /// Returns reviews for a specific product (nested resource).
    [HttpGet("{productId:int}/reviews")]
    public async Task<ActionResult<IEnumerable<ReviewDto>>> GetReviews(int productId)
    {
        var reviews = await _service.GetReviewsAsync(productId);
        return Ok(reviews);
    }

    /// POST api/Products
    /// Creates a new product.
    [HttpPost]
    [Authorize(Roles = "Admin")]
    public async Task<ActionResult<ProductDto>> Create(CreateProductDto dto)
    {
        var product = await _service.CreateAsync(dto);

        // Returns 201 with Location: api/Products/{id}
        return CreatedAtRoute("GetProduct", new { id = product.Id }, product);
    }

    /// PUT api/Products/5
    /// Full update of an existing product.
    [HttpPut("{id:int}")]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> Update(int id, UpdateProductDto dto)
    {
        var exists = await _service.ExistsAsync(id);
        if (!exists) return NotFound();

        await _service.UpdateAsync(id, dto);
        return NoContent();  // 204
    }

    /// PATCH api/Products/5
    /// Partial update of an existing product.
    [HttpPatch("{id:int}")]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> PartialUpdate(
        int id,
        [FromBody] JsonPatchDocument<UpdateProductDto> patchDoc)
    {
        var product = await _service.GetByIdAsync(id);
        if (product is null) return NotFound();

        // Apply and validate the patch
        var dto = new UpdateProductDto(); // map from product
        patchDoc.ApplyTo(dto, ModelState);
        if (!ModelState.IsValid) return BadRequest(ModelState);

        await _service.UpdateAsync(id, dto);
        return NoContent();
    }

    /// DELETE api/Products/5
    /// Deletes a product.
    [HttpDelete("{id:int}")]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> Delete(int id)
    {
        var exists = await _service.ExistsAsync(id);
        if (!exists) return NotFound();

        await _service.DeleteAsync(id);
        return NoContent();  // 204
    }
}
```

### What This Controller Demonstrates

| Feature | Where Used |
|---|---|
| Controller prefix with `[controller]` token | `[Route("api/[controller]")]` |
| HTTP method attributes with templates | `[HttpGet("{id:int}")]`, `[HttpPost]`, etc. |
| Route constraints | `{id:int}`, `{slug:regex(...)}` |
| Named routes | `Name = "GetProduct"` |
| `CreatedAtRoute` for 201 responses | `Create` action |
| Nested resources | `{productId:int}/reviews` |
| Authorization on specific actions | `[Authorize(Roles = "Admin")]` |
| Query string parameters with `[FromQuery]` | `GetAll` action |

> [!summary] Section Summary
> - A well-designed RESTful controller uses `[Route("api/[controller]")]` as a prefix.
> - Each CRUD operation maps to an HTTP method: GET (read), POST (create), PUT (full update), PATCH (partial), DELETE.
> - Named routes enable `CreatedAtRoute` for proper 201 responses.
> - Route constraints (`{id:int}`) prevent invalid values from reaching action code.
> - Nested resources use compound templates like `{productId:int}/reviews`.
