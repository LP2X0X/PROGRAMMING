---
tags:
  - csharp
  - asp-net-core
  - web-api
  - controllers
---


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
