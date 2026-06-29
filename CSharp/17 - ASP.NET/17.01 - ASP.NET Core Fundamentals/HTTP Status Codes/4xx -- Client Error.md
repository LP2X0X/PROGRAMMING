---
tags: [csharp, asp-net-core, http, status-codes, web-api, fundamentals]
---


**Client error responses** mean ==the problem is on the client's side==. The request was malformed, unauthorized, or asked for something that does not exist. The client needs to fix the request before trying again.

This is the largest and most important class for API developers.

| Code | Name | Meaning | ASP.NET Core Helper |
|---|---|---|---|
| **400** | Bad Request | The request is malformed or contains invalid data. | `BadRequest()` / `BadRequest(modelState)` |
| **401** | Unauthorized | The client is ==not authenticated== (no credentials or invalid credentials). | `Unauthorized()` / `Challenge()` |
| **403** | Forbidden | The client is ==authenticated but not authorized== to access this resource. | `Forbid()` |
| **404** | Not Found | The requested resource does not exist. | `NotFound()` / `NotFound(value)` |
| **405** | Method Not Allowed | The HTTP method is not supported for this URL (e.g., POST to a GET-only endpoint). | Returned automatically by routing |
| **409** | Conflict | The request conflicts with the current state of the resource (e.g., duplicate key, concurrency conflict). | `Conflict()` / `Conflict(value)` |
| **415** | Unsupported Media Type | The `Content-Type` header is not supported (e.g., sending XML when the API only accepts JSON). | Returned automatically by `[ApiController]` |
| **422** | Unprocessable Entity | The request syntax is correct but the data is semantically invalid (e.g., end date before start date). | `UnprocessableEntity()` |
| **429** | Too Many Requests | Rate limiting -- the client has sent too many requests. | Returned by rate limiting middleware |

#### 400 Bad Request -- Invalid Input

The most general "your request is wrong" code. Use it for validation errors, missing required fields, and malformed input.

```csharp
[HttpPost]
public async Task<IActionResult> Create(CreateProductDto dto)
{
    // Automatic model validation with [ApiController] attribute
    // returns 400 with validation errors automatically

    // Manual validation example:
    if (dto.Price < 0)
        return BadRequest(new { error = "Price cannot be negative" });

    var product = await _service.CreateAsync(dto);
    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
}
```

> [!ad-tip] Automatic 400 with `[ApiController]`
> When your controller has the `[ApiController]` attribute, ASP.NET Core ==automatically returns 400== with a [[Problem Details]] body if model validation fails. You do not need to check `ModelState.IsValid` manually.
> ```csharp
> [ApiController] // Enables automatic 400 for invalid models
> [Route("api/[controller]")]
> public class ProductsController : ControllerBase
> {
>     [HttpPost]
>     public async Task<IActionResult> Create(
>         CreateProductDto dto) // Invalid model → automatic 400
>     {
>         // If we reach here, the model is valid
>         var product = await _service.CreateAsync(dto);
>         return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
>     }
> }
> ```

#### 401 Unauthorized vs 403 Forbidden

This is one of the most commonly confused pairs in HTTP. The naming is misleading -- `401` is about ==authentication==, not authorization.

```csharp
// 401 -- "I don't know who you are" (not authenticated)
// Missing or invalid credentials (no token, expired token, bad password)
[HttpGet("profile")]
[Authorize] // Returns 401 if no valid auth token is present
public IActionResult GetProfile()
{
    return Ok(new { name = User.Identity?.Name });
}

// 403 -- "I know who you are, but you can't do this" (not authorized)
// Valid credentials, but insufficient permissions
[HttpDelete("{id}")]
[Authorize(Roles = "Admin")] // Returns 403 if user is authenticated but not Admin
public async Task<IActionResult> Delete(int id)
{
    await _service.DeleteAsync(id);
    return NoContent();
}
```

> [!ad-warning] 401 vs 403 -- The Naming Is Misleading
> Despite its name, **401 "Unauthorized"** actually means **"Unauthenticated"** -- the server does not know who you are.
> **403 "Forbidden"** means **"Unauthorized"** -- the server knows who you are but you do not have permission.
>
> | Situation | Desktop Equivalent | HTTP Code |
> |---|---|---|
> | No login token provided | `NullReferenceException` on user context | **401** |
> | Expired or invalid token | `SecurityTokenExpiredException` | **401** |
> | Valid token, wrong role | `UnauthorizedAccessException` | **403** |
>
> In ASP.NET Core, the `[Authorize]` attribute returns 401 when there is no valid identity, and the `[Authorize(Roles = "Admin")]` attribute returns 403 when the identity exists but lacks the required role.

#### 404 Not Found -- Resource Does Not Exist

```csharp
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null)
        return NotFound(); // 404

    return Ok(product); // 200
}

// With a message body:
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null)
        return NotFound(new { message = $"Product with ID {id} was not found." });

    return Ok(product);
}
```

#### 409 Conflict -- State Conflict

Used when the request would cause an inconsistency. Common for duplicate records, concurrency conflicts, or state machine violations.

```csharp
[HttpPost]
public async Task<IActionResult> Create(CreateUserDto dto)
{
    if (await _repository.EmailExistsAsync(dto.Email))
        return Conflict(new { error = $"A user with email '{dto.Email}' already exists." });

    var user = await _service.CreateAsync(dto);
    return CreatedAtAction(nameof(GetById), new { id = user.Id }, user);
}

// Concurrency conflict example (optimistic concurrency with EF Core)
[HttpPut("{id}")]
public async Task<IActionResult> Update(int id, UpdateProductDto dto)
{
    try
    {
        await _service.UpdateAsync(id, dto);
        return NoContent();
    }
    catch (DbUpdateConcurrencyException)
    {
        return Conflict(new { error = "The product was modified by another user. Please refresh and try again." });
    }
}
```

#### 422 Unprocessable Entity -- Semantically Invalid

The request is syntactically correct (JSON is valid, all required fields present) but the data does not make sense logically.

```csharp
[HttpPost("reservations")]
public async Task<IActionResult> CreateReservation(ReservationDto dto)
{
    // Model binding and basic validation pass (no 400 from [ApiController])
    // But business rules fail:
    if (dto.EndDate <= dto.StartDate)
    {
        return UnprocessableEntity(new
        {
            error = "End date must be after start date."
        });
    }

    var reservation = await _service.CreateAsync(dto);
    return CreatedAtAction(nameof(GetById), new { id = reservation.Id }, reservation);
}
```

> [!ad-tip] 400 vs 422 -- When to Use Which
> - **400**: The request is structurally wrong -- missing fields, wrong types, malformed JSON
> - **422**: The request structure is fine but the *business logic* rejects the data -- end date before start date, negative quantity, self-referencing parent
>
> In practice, many APIs use 400 for both scenarios. Using 422 for semantic validation is more precise, but either is acceptable as long as you are consistent.

> [!summary] Section Summary
> - 4xx codes mean the client sent a bad request and needs to fix it
> - 400: malformed or invalid data; returned automatically by `[ApiController]` for model validation failures
> - 401: not authenticated (no credentials or invalid credentials) -- despite the misleading name "Unauthorized"
> - 403: authenticated but not authorized (insufficient permissions)
> - 404: resource does not exist
> - 409: request conflicts with current state (duplicates, concurrency)
> - 422: syntactically valid but semantically wrong (business rule violations)
> - The 401 vs 403 distinction is one of the most commonly confused concepts in HTTP
