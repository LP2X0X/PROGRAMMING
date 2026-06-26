---
tags:
 - csharp
 - asp-net-core
 - model-binding
 - controllers
---

# Model Binding

```ad-note
Model binding is the mechanism ASP.NET Core uses to automatically extract data from HTTP requests and map it to action method parameters and model properties. It eliminates the need for manual parsing of query strings, form data, route values, and request bodies -- allowing you to write action methods that receive strongly-typed objects directly.
```

---

## Table of Contents

- [What Model Binding Is](#What%20Model%20Binding%20Is)
- [Binding Sources (Default Priority Order)](#Binding%20Sources%20(Default%20Priority%20Order))
- [Explicit Binding Source Attributes](#Explicit%20Binding%20Source%20Attributes)
- [Simple Type Binding](#Simple%20Type%20Binding)
- [Complex Type Binding](#Complex%20Type%20Binding)
- [ApiController Changes to Default Behavior](#ApiController%20Changes%20to%20Default%20Behavior)
- [Collection Binding](#Collection%20Binding)
- [File Upload Binding](#File%20Upload%20Binding)
- [Custom Model Binders](#Custom%20Model%20Binders)
- [BindProperty and BindProperties](#BindProperty%20and%20BindProperties)
- [Bind and BindNever -- Controlling What Gets Bound](#Bind%20and%20BindNever%20--%20Controlling%20What%20Gets%20Bound)
- [Real-World Example](#Real-World%20Example)
- [Comprehensive Summary](#Comprehensive%20Summary)
- [Related Topics](#Related%20Topics)

---

## What Model Binding Is

When a controller action method declares parameters, ASP.NET Core does not force you to dig through the raw HTTP request manually. Instead, the **model binding** system inspects the incoming request and automatically populates those parameters with values from various parts of the request.

Without model binding, you would write code like this:

```csharp
[HttpGet]
public IActionResult Search()
{
    // Manually extracting values -- tedious and error-prone
    string term = Request.Query["term"];
    int page = int.Parse(Request.Query["page"]);
    string sort = Request.Query["sort"];
    
    // ... use values
}
```

With model binding, this becomes:

```csharp
[HttpGet]
public IActionResult Search(string term, int page, string sort)
{
    // Values are already extracted and converted
    // ... use values directly
}
```

### How It Fits in the Request Pipeline

The model binding system runs **after** routing selects the action method but **before** the action method executes. The sequence is:

1. Routing matches the request to a controller action (see [[17.05 - Routing]])
2. **Model binding** extracts values from the request and populates action parameters
3. **[[Validation]]** runs on the bound model (both data annotations and custom validators)
4. The action method executes with the bound parameters

### What Happens When Binding Fails

If the binder cannot convert a request value to the target type, two things happen:

- The parameter receives its **default value** (`null` for reference types, `0` for `int`, `false` for `bool`, etc.)
- An error is recorded in `ModelState`, which the action can inspect

```csharp
[HttpGet("products/{id}")]
public IActionResult GetProduct(int id)
{
    // If someone requests /products/abc, binding fails:
    // - id will be 0 (default for int)
    // - ModelState.IsValid will be false
    // - ModelState["id"].Errors will contain the conversion error
    
    if (!ModelState.IsValid)
        return BadRequest(ModelState);
    
    // ... proceed with valid id
}
```

```ad-info
With `[ApiController]`, invalid `ModelState` automatically returns a 400 Bad Request response before the action method even runs. You do not need to check `ModelState.IsValid` manually. See [[Validation]] for details.
```

---

## Binding Sources (Default Priority Order)

Model binding searches for values in a specific order. When a parameter name matches a value in multiple sources, the **first source that produces a match wins**.

The default priority order for non-`[ApiController]` controllers is:

| Priority | Source | Content Type | Included by Default |
|----------|--------|-------------|-------------------|
| 1 | Form values | `application/x-www-form-urlencoded` or `multipart/form-data` | Yes |
| 2 | Route values | URL path segments | Yes |
| 3 | Query string | URL parameters after `?` | Yes |
| 4 | Header values | HTTP headers | No (requires `[FromHeader]`) |
| 5 | Body | JSON/XML payload | No (requires `[FromBody]` or `[ApiController]`) |

### 1. Form Values

Form data comes from HTML form submissions with POST method. The content type is either `application/x-www-form-urlencoded` (standard forms) or `multipart/form-data` (forms with file uploads).

```csharp
// HTML form POSTs name=Widget&price=29.99
[HttpPost]
public IActionResult CreateProduct(string name, decimal price)
{
    // name = "Widget", price = 29.99
    // Bound from form values (highest priority)
}
```

### 2. Route Values

Route values come from URL segments matched by route template parameters:

```csharp
[HttpGet("products/{id}/reviews/{reviewId}")]
public IActionResult GetReview(int id, int reviewId)
{
    // GET /products/5/reviews/42
    // id = 5, reviewId = 42
    // Bound from route values
}
```

### 3. Query String

Query string values come from URL parameters after the `?`:

```csharp
[HttpGet("products")]
public IActionResult Search(string term, int page, string sort)
{
    // GET /products?term=laptop&page=2&sort=price
    // term = "laptop", page = 2, sort = "price"
    // Bound from query string
}
```

### 4. Header Values (Not Default)

Headers are **not** searched by default. You must explicitly use `[FromHeader]`:

```csharp
[HttpGet("products")]
public IActionResult List([FromHeader(Name = "X-Correlation-Id")] string correlationId)
{
    // Bound from the X-Correlation-Id HTTP header
}
```

### 5. Body (Not Default)

The request body is **not** searched by default in regular controllers. You must use `[FromBody]`:

```csharp
[HttpPost("products")]
public IActionResult Create([FromBody] CreateProductRequest request)
{
    // Bound from JSON request body
}
```

### Priority Resolution Example

When the same parameter name exists in multiple sources, priority determines which value wins:

```csharp
// Route template: products/{id}
[HttpPost("products/{id}")]
public IActionResult Update(int id, string name)
{
    // Request: POST /products/5?id=99
    // Form body also contains: id=200
    
    // What value does 'id' get?
    // 1. Form values checked first -> id=200 found -> id = 200
    // (Route value of 5 and query string value of 99 are ignored)
    
    // To be explicit and avoid confusion, use source attributes:
}

// Better -- explicit about where each value comes from:
[HttpPost("products/{id}")]
public IActionResult Update([FromRoute] int id, [FromBody] UpdateProductRequest request)
{
    // id always comes from route, request always from body
}
```

```ad-tip
Always use explicit binding source attributes when an action method accepts values from multiple sources. Relying on implicit priority ordering makes the code harder to reason about and can lead to subtle bugs.
```

---

## Explicit Binding Source Attributes

Explicit binding source attributes override the default priority order and tell the binder exactly where to look for a value.

### [FromBody]

Binds a parameter from the **request body** using the configured input formatter (JSON by default).

```csharp
[HttpPost("orders")]
public IActionResult CreateOrder([FromBody] CreateOrderRequest request)
{
    // Expects JSON body:
    // {
    //     "customerId": 42,
    //     "items": [
    //         { "productId": 1, "quantity": 2 },
    //         { "productId": 3, "quantity": 1 }
    //     ]
    // }
}
```

```ad-warning
Only **one** parameter per action method can use `[FromBody]`. The request body is a forward-only stream that can only be read once. If you need both body data and route data, use `[FromBody]` for the body and `[FromRoute]` for the route parameter.
```

### [FromRoute]

Binds a parameter exclusively from route data:

```csharp
[HttpPut("products/{id}")]
public IActionResult Update([FromRoute] int id, [FromBody] UpdateProductRequest request)
{
    // id is guaranteed to come from the URL path segment
    // PUT /products/5 -> id = 5
}
```

### [FromQuery]

Binds a parameter exclusively from the query string:

```csharp
[HttpGet("products")]
public IActionResult Search(
    [FromQuery] string term,
    [FromQuery] int page = 1,
    [FromQuery] int pageSize = 20,
    [FromQuery] string sortBy = "name")
{
    // GET /products?term=laptop&page=2&pageSize=50&sortBy=price
}
```

You can specify a different query parameter name than the C# parameter name:

```csharp
[HttpGet("products")]
public IActionResult Search([FromQuery(Name = "q")] string searchTerm)
{
    // GET /products?q=laptop -> searchTerm = "laptop"
}
```

### [FromHeader]

Binds a parameter from an HTTP request header:

```csharp
[HttpGet("products")]
public IActionResult List(
    [FromHeader(Name = "Accept-Language")] string language,
    [FromHeader(Name = "X-Request-Id")] string requestId)
{
    // language bound from Accept-Language header
    // requestId bound from X-Request-Id header
}
```

```ad-info
HTTP headers use **hyphens** (`Accept-Language`, `X-Request-Id`) while C# properties use **PascalCase**. When binding to a property name without specifying `Name`, the binder converts PascalCase to hyphen-case automatically. For example, a property named `AcceptLanguage` would match the header `Accept-Language`. However, explicitly specifying the `Name` parameter is clearer and recommended.
```

### [FromForm]

Binds a parameter from form data, useful when you want to be explicit or when a method might otherwise default to body binding:

```csharp
[HttpPost("products")]
public IActionResult Create([FromForm] CreateProductRequest request)
{
    // Binds from form fields, NOT from JSON body
    // Useful in API controllers where complex types default to [FromBody]
}
```

### [FromServices]

Binds a parameter from the **dependency injection container** rather than from request data:

```csharp
[HttpGet("products/{id}/report")]
public IActionResult GenerateReport(
    [FromRoute] int id,
    [FromServices] IReportGenerator reportGenerator)
{
    // reportGenerator is resolved from DI, not from the request
    // Useful for services only needed in one action,
    // avoiding constructor injection bloat
    var report = reportGenerator.Generate(id);
    return File(report, "application/pdf");
}
```

```ad-tip
Use `[FromServices]` sparingly. If a service is used in multiple actions, inject it through the constructor instead. `[FromServices]` is best for services used in only one or two action methods.
```

---

## Simple Type Binding

Simple types are scalar values: `int`, `string`, `bool`, `DateTime`, `Guid`, `decimal`, `double`, `enum`, and others that have a `TypeConverter` capable of converting from a string.

### How Conversion Works

All HTTP request data arrives as strings. The model binder uses `TypeConverter` to convert the string representation to the target CLR type:

```csharp
[HttpGet("products")]
public IActionResult Search(
    string name,           // No conversion needed -- already a string
    int page,              // "2" -> 2
    decimal minPrice,      // "19.99" -> 19.99m
    bool inStock,          // "true" -> true (also accepts "True", "TRUE")
    DateTime since,        // "2024-01-15" -> DateTime(2024, 1, 15)
    Guid correlationId,    // "a1b2c3d4-..." -> Guid
    SortOrder sort)        // "Descending" -> SortOrder.Descending (enum)
{
    // All parameters automatically converted from query string values
}

public enum SortOrder
{
    Ascending,
    Descending
}
```

### Special Types

Some types receive special treatment from the binder:

```csharp
[HttpGet("products")]
public async Task<IActionResult> Search(
    string term,
    CancellationToken cancellationToken)  // Automatically bound to HttpContext.RequestAborted
{
    var products = await _productService.SearchAsync(term, cancellationToken);
    return Ok(products);
}
```

`CancellationToken` parameters are automatically bound to `HttpContext.RequestAborted` -- they do not come from request data.

### Conversion Failures

When a string value cannot be converted to the target type, the binder records the error:

```csharp
[HttpGet("products/{id}")]
public IActionResult GetProduct(int id)
{
    // GET /products/abc
    // Conversion from "abc" to int fails:
    // - id = 0 (default)
    // - ModelState["id"].Errors contains:
    //   "The value 'abc' is not valid for id."
}
```

### Nullable Types

Nullable types allow binding to succeed even when the value is absent:

```csharp
[HttpGet("products")]
public IActionResult Search(
    string? term,          // null if not provided
    int? page,             // null if not provided (vs. 0 for non-nullable)
    decimal? maxPrice)     // null if not provided
{
    int actualPage = page ?? 1;        // Default to 1 if not provided
    // term can be checked for null to determine if filtering is requested
}
```

---

## Complex Type Binding

Complex types are classes or records with multiple properties. The binder creates an instance and recursively sets each property by matching property names to request values.

### Basic Complex Type

```csharp
public class ProductSearchCriteria
{
    public string? Term { get; set; }
    public int Page { get; set; } = 1;
    public int PageSize { get; set; } = 20;
    public string SortBy { get; set; } = "name";
    public bool Descending { get; set; }
}

[HttpGet("products")]
public IActionResult Search(ProductSearchCriteria criteria)
{
    // GET /products?term=laptop&page=2&pageSize=50&sortBy=price&descending=true
    // 
    // criteria.Term = "laptop"
    // criteria.Page = 2
    // criteria.PageSize = 50
    // criteria.SortBy = "price"
    // criteria.Descending = true
}
```

### Property Name Matching

Property name matching is **case-insensitive**. The request value `pagesize`, `PageSize`, and `PAGESIZE` all match a property named `PageSize`.

### Nested Objects

Complex types can contain nested objects. The binder uses **dot notation** in form field names or query parameters:

```csharp
public class CreateCustomerRequest
{
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public Address ShippingAddress { get; set; } = new();
}

public class Address
{
    public string Street { get; set; } = string.Empty;
    public string City { get; set; } = string.Empty;
    public string State { get; set; } = string.Empty;
    public string ZipCode { get; set; } = string.Empty;
}

[HttpPost("customers")]
public IActionResult CreateCustomer(CreateCustomerRequest request)
{
    // Form fields or query parameters use dot notation:
    // Name=John+Doe
    // Email=john@example.com
    // ShippingAddress.Street=123+Main+St
    // ShippingAddress.City=Springfield
    // ShippingAddress.State=IL
    // ShippingAddress.ZipCode=62704
}
```

### Records

C# records work with model binding if they have a parameterless constructor or settable properties. The most compatible pattern uses `init` properties:

```csharp
public record ProductFilter
{
    public string? Term { get; init; }
    public decimal? MinPrice { get; init; }
    public decimal? MaxPrice { get; init; }
    public string? Category { get; init; }
}

[HttpGet("products")]
public IActionResult Search([FromQuery] ProductFilter filter)
{
    // GET /products?term=laptop&minPrice=500&maxPrice=1500&category=electronics
}
```

```ad-note
The binder needs to be able to create an instance of the type first, then set properties. Classes and records must have a **public parameterless constructor** (or the binder must be able to resolve one). Primary constructor records like `record ProductFilter(string Term, int Page)` can work but require all values to be present -- missing values cause binding failures rather than defaults.
```

---

## ApiController Changes to Default Behavior

The `[ApiController]` attribute (applied to controllers inheriting from `ControllerBase`) changes the default binding source inference rules significantly.

### Default Inference Rules with [ApiController]

| Parameter Type | Inferred Source |
|---------------|----------------|
| Complex types (classes, records) | `[FromBody]` (JSON) |
| Simple types matching a route parameter | `[FromRoute]` |
| Simple types not matching a route parameter | `[FromQuery]` |
| `IFormFile`, `IFormFileCollection` | `[FromForm]` |
| `CancellationToken` | `HttpContext.RequestAborted` |

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // Complex type -> automatically [FromBody]
    [HttpPost]
    public IActionResult Create(CreateProductRequest request)
    {
        // No [FromBody] needed -- [ApiController] infers it
        // Expects JSON body
    }
    
    // Simple type matching route param -> [FromRoute]
    // Simple type not in route -> [FromQuery]
    [HttpGet("{id}")]
    public IActionResult Get(int id, bool includeReviews)
    {
        // id -> [FromRoute] (matches {id} in route template)
        // includeReviews -> [FromQuery] (no matching route param)
        // GET /api/products/5?includeReviews=true
    }
}
```

### Why API Controllers "Just Work" with JSON

This is why you can send JSON to an API controller action without explicitly adding `[FromBody]`:

```csharp
[ApiController]
[Route("api/orders")]
public class OrdersController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(CreateOrderRequest request)
    {
        // This "just works" with JSON because:
        // 1. CreateOrderRequest is a complex type
        // 2. [ApiController] infers [FromBody] for complex types
        // 3. The default input formatter is System.Text.Json
        return CreatedAtAction(nameof(Get), new { id = request.Id }, request);
    }
    
    [HttpGet("{id}")]
    public IActionResult Get(int id) => Ok();
}
```

### Disabling Inference

If you need to opt out of the automatic inference for a specific parameter, apply an explicit binding source attribute:

```csharp
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    // Override: bind complex type from form instead of body
    [HttpPost]
    public IActionResult Create([FromForm] CreateProductRequest request)
    {
        // Even though [ApiController] would infer [FromBody],
        // the explicit [FromForm] takes precedence
    }
}
```

```ad-attention
When using `[ApiController]`, if you have an action that receives **both** a complex type from the body and a file upload, you need to explicitly mark the complex type as `[FromForm]` because you cannot mix `[FromBody]` and `[FromForm]` -- multipart forms carry both data and files together.
```

---

## Collection Binding

Model binding supports arrays, lists, and dictionaries from both query strings and form data.

### Arrays and Lists from Query String

There are two syntaxes for binding collections from query strings:

**Repeated parameter names (preferred):**

```csharp
[HttpGet("products")]
public IActionResult GetByIds([FromQuery] List<int> ids)
{
    // GET /products?ids=1&ids=2&ids=3
    // ids = [1, 2, 3]
}
```

**Indexed syntax:**

```csharp
[HttpGet("products")]
public IActionResult GetByIds([FromQuery] List<int> ids)
{
    // GET /products?ids[0]=1&ids[1]=2&ids[2]=3
    // ids = [1, 2, 3]
    // Both syntaxes produce the same result
}
```

### Dictionaries from Query String

```csharp
[HttpGet("products")]
public IActionResult Search([FromQuery] Dictionary<string, string> filters)
{
    // GET /products?filters[category]=electronics&filters[brand]=acme&filters[inStock]=true
    //
    // filters = {
    //     ["category"] = "electronics",
    //     ["brand"] = "acme",
    //     ["inStock"] = "true"
    // }
}
```

### Lists of Complex Types from Form Data

Form data can bind collections of complex objects using indexed property names:

```csharp
public class OrderItemRequest
{
    public int ProductId { get; set; }
    public string ProductName { get; set; } = string.Empty;
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

[HttpPost("orders")]
public IActionResult CreateOrder(List<OrderItemRequest> items)
{
    // Form data:
    // items[0].ProductId=1
    // items[0].ProductName=Widget
    // items[0].Quantity=3
    // items[0].UnitPrice=9.99
    // items[1].ProductId=2
    // items[1].ProductName=Gadget
    // items[1].Quantity=1
    // items[1].UnitPrice=24.99
    
    // items = [
    //     { ProductId=1, ProductName="Widget", Quantity=3, UnitPrice=9.99 },
    //     { ProductId=2, ProductName="Gadget", Quantity=1, UnitPrice=24.99 }
    // ]
}
```

```ad-warning
When binding lists of complex types, the indices **must be sequential starting from 0**. If you skip an index (e.g., `items[0]` then `items[2]`), the binder stops at the gap and only binds `items[0]`.
```

---

## File Upload Binding

ASP.NET Core provides built-in support for binding uploaded files through the `IFormFile` and `IFormFileCollection` interfaces.

### Single File Upload

```csharp
[HttpPost("products/{id}/image")]
public async Task<IActionResult> UploadImage(
    [FromRoute] int id,
    IFormFile image)
{
    if (image == null || image.Length == 0)
        return BadRequest("No file uploaded.");
    
    // Available properties:
    // image.FileName       -> original file name from the client
    // image.Length          -> file size in bytes
    // image.ContentType    -> MIME type (e.g., "image/png")
    // image.Name           -> the form field name
    
    // Size validation
    const long maxSize = 5 * 1024 * 1024; // 5 MB
    if (image.Length > maxSize)
        return BadRequest($"File exceeds maximum size of {maxSize / 1024 / 1024} MB.");
    
    // Content type validation
    var allowedTypes = new[] { "image/jpeg", "image/png", "image/webp" };
    if (!allowedTypes.Contains(image.ContentType))
        return BadRequest("Only JPEG, PNG, and WebP images are allowed.");
    
    // Save to disk
    var filePath = Path.Combine("uploads", $"{id}_{Guid.NewGuid()}{Path.GetExtension(image.FileName)}");
    
    using (var stream = new FileStream(filePath, FileMode.Create))
    {
        await image.CopyToAsync(stream);
    }
    
    return Ok(new { path = filePath });
}
```

### Multiple File Uploads

```csharp
[HttpPost("products/{id}/photos")]
public async Task<IActionResult> UploadPhotos(
    [FromRoute] int id,
    IFormFileCollection photos)  // or List<IFormFile> photos
{
    if (photos.Count == 0)
        return BadRequest("No files uploaded.");
    
    var uploadedPaths = new List<string>();
    
    foreach (var photo in photos)
    {
        if (photo.Length > 0)
        {
            var filePath = Path.Combine("uploads", $"{id}_{Guid.NewGuid()}{Path.GetExtension(photo.FileName)}");
            using var stream = new FileStream(filePath, FileMode.Create);
            await photo.CopyToAsync(stream);
            uploadedPaths.Add(filePath);
        }
    }
    
    return Ok(new { count = uploadedPaths.Count, paths = uploadedPaths });
}
```

### The HTML Form

The form **must** use `enctype="multipart/form-data"` for file uploads to work:

```html
<form method="post"
      action="/products/5/photos"
      enctype="multipart/form-data">
    
    <input type="file" name="photos" multiple />
    <button type="submit">Upload Photos</button>
</form>
```

```ad-danger
Never trust the file name or content type from the client. Always validate and sanitize:
- Generate a new file name server-side (do not use the client-provided name directly for storage)
- Validate the content type and file extension
- Enforce size limits
- Consider scanning for malware in production systems
- Do not save files to a location within the web root where they could be executed
```

---

## Custom Model Binders

When the built-in binders cannot handle your scenario, you can implement a custom model binder by implementing the `IModelBinder` interface.

### Use Case: Comma-Separated Values to List

A common need is binding a comma-separated query string value like `?ids=1,2,3` into a `List<int>`:

```csharp
using Microsoft.AspNetCore.Mvc.ModelBinding;

public class CommaSeparatedModelBinder : IModelBinder
{
    public Task BindModelAsync(ModelBindingContext bindingContext)
    {
        ArgumentNullException.ThrowIfNull(bindingContext);
        
        var modelName = bindingContext.ModelName;
        var valueProviderResult = bindingContext.ValueProvider.GetValue(modelName);
        
        if (valueProviderResult == ValueProviderResult.None)
            return Task.CompletedTask;
        
        bindingContext.ModelState.SetModelValue(modelName, valueProviderResult);
        
        var value = valueProviderResult.FirstValue;
        
        if (string.IsNullOrWhiteSpace(value))
            return Task.CompletedTask;
        
        // Split the comma-separated string and parse each element
        var items = new List<int>();
        foreach (var segment in value.Split(',', StringSplitOptions.RemoveEmptyEntries))
        {
            if (int.TryParse(segment.Trim(), out int parsed))
            {
                items.Add(parsed);
            }
            else
            {
                bindingContext.ModelState.TryAddModelError(
                    modelName,
                    $"'{segment.Trim()}' is not a valid integer.");
                return Task.CompletedTask;
            }
        }
        
        bindingContext.Result = ModelBindingResult.Success(items);
        return Task.CompletedTask;
    }
}
```

### Applying the Custom Binder

There are several ways to register a custom binder.

**Per-parameter with `[ModelBinder]`:**

```csharp
[HttpGet("products")]
public IActionResult GetByIds(
    [ModelBinder(BinderType = typeof(CommaSeparatedModelBinder))] List<int> ids)
{
    // GET /products?ids=1,2,3
    // ids = [1, 2, 3]
}
```

**Via a custom attribute:**

```csharp
[AttributeUsage(AttributeTargets.Parameter | AttributeTargets.Property)]
public class CommaSeparatedAttribute : ModelBinderAttribute
{
    public CommaSeparatedAttribute()
        : base(typeof(CommaSeparatedModelBinder))
    {
    }
}

// Usage becomes cleaner:
[HttpGet("products")]
public IActionResult GetByIds([CommaSeparated] List<int> ids)
{
    // GET /products?ids=1,2,3
}
```

**Globally via a model binder provider:**

```csharp
public class CommaSeparatedModelBinderProvider : IModelBinderProvider
{
    public IModelBinder? GetBinder(ModelBinderProviderContext context)
    {
        ArgumentNullException.ThrowIfNull(context);
        
        // Apply this binder to any List<int> parameter decorated with [CommaSeparated]
        if (context.Metadata.ModelType == typeof(List<int>)
            && context.Metadata is Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.DefaultModelMetadata metadata
            && metadata.Attributes.ParameterAttributes?.OfType<CommaSeparatedAttribute>().Any() == true)
        {
            return new CommaSeparatedModelBinder();
        }
        
        return null;
    }
}

// Register in Program.cs:
builder.Services.AddControllers(options =>
{
    options.ModelBinderProviders.Insert(0, new CommaSeparatedModelBinderProvider());
});
```

```ad-note
Custom binder providers are checked in order. Insert your provider at position 0 to ensure it is evaluated before the built-in providers. If your provider returns `null`, the next provider in the list is tried.
```

---

## BindProperty and BindProperties

While action method parameters are bound automatically, **controller properties** and **Razor Pages PageModel properties** are not bound by default. The `[BindProperty]` and `[BindProperties]` attributes opt in to property-level binding.

### [BindProperty] on Individual Properties

```csharp
public class ProductsController : Controller
{
    [BindProperty]
    public string SearchTerm { get; set; } = string.Empty;
    
    [BindProperty]
    public int Page { get; set; } = 1;
    
    [HttpPost]
    public IActionResult Search()
    {
        // SearchTerm and Page are bound from the request on POST
        var results = _service.Search(SearchTerm, Page);
        return View(results);
    }
}
```

### [BindProperties] on the Class

```csharp
[BindProperties]
public class ProductsController : Controller
{
    public string SearchTerm { get; set; } = string.Empty;
    public int Page { get; set; } = 1;
    public string SortBy { get; set; } = "name";
    
    // ALL public properties are bound from the request
}
```

### GET Requests and SupportsGet

By default, `[BindProperty]` and `[BindProperties]` only bind on **POST, PUT, PATCH, and DELETE** requests -- not on GET. To enable binding on GET requests, set `SupportsGet = true`:

```csharp
public class ProductsController : Controller
{
    [BindProperty(SupportsGet = true)]
    public string? SearchTerm { get; set; }
    
    [BindProperty(SupportsGet = true)]
    public int Page { get; set; } = 1;
    
    [HttpGet]
    public IActionResult Search()
    {
        // GET /products?searchTerm=laptop&page=2
        // SearchTerm and Page are now bound from query string on GET
    }
}
```

### Razor Pages PageModel Example

`[BindProperty]` is especially common in Razor Pages, where the PageModel serves as both the handler and the view model:

```csharp
public class CreateProductModel : PageModel
{
    private readonly IProductService _productService;
    
    public CreateProductModel(IProductService productService)
    {
        _productService = productService;
    }
    
    [BindProperty]
    public ProductInputModel Input { get; set; } = new();
    
    // Display data -- not bound from request (no [BindProperty])
    public List<string> Categories { get; set; } = new();
    
    public void OnGet()
    {
        Categories = _productService.GetCategories();
    }
    
    public async Task<IActionResult> OnPostAsync()
    {
        // Input is automatically bound from the form on POST
        if (!ModelState.IsValid)
        {
            Categories = _productService.GetCategories();
            return Page();
        }
        
        await _productService.CreateAsync(Input);
        return RedirectToPage("./Index");
    }
}

public class ProductInputModel
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; } = string.Empty;
    
    [Range(0.01, 99999.99)]
    public decimal Price { get; set; }
    
    public string? Description { get; set; }
    public string? Category { get; set; }
}
```

```ad-attention
Be cautious with `[BindProperties]` on controller classes -- it binds **all** public properties, which could expose properties you did not intend to be settable from request data. Prefer `[BindProperty]` on individual properties for tighter control.
```

---

## Bind and BindNever -- Controlling What Gets Bound

These attributes control which properties of a model are eligible for binding, primarily as a defense against **over-posting** (mass assignment) attacks.

### The Over-Posting Problem

Over-posting occurs when a malicious user sends extra form fields that match properties on your model that you did not intend to expose:

```csharp
public class UserProfile
{
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public bool IsAdmin { get; set; }   // Should NEVER be set by user input
}

[HttpPost("profile")]
public IActionResult UpdateProfile(UserProfile profile)
{
    // A malicious user sends: Name=Hacker&Email=h@evil.com&IsAdmin=true
    // Without protection, profile.IsAdmin = true!
    _repository.Update(profile);
    return Ok();
}
```

```ad-danger
Over-posting is a real security vulnerability. A classic example is the 2012 GitHub incident where a user exploited mass assignment to add their SSH key to any repository. Always limit which properties can be bound from request data.
```

### [Bind] -- Allow-List Approach

`[Bind]` specifies which properties **are** allowed to be bound. All other properties are ignored:

```csharp
[HttpPost("profile")]
public IActionResult UpdateProfile([Bind("Name,Email")] UserProfile profile)
{
    // Only Name and Email are bound from the request
    // IsAdmin is always false (default) regardless of what the client sends
    
    _repository.Update(profile);
    return Ok();
}
```

### [BindNever] -- Deny-List Approach

`[BindNever]` is applied to model properties to exclude them from binding:

```csharp
using Microsoft.AspNetCore.Mvc.ModelBinding;

public class UserProfile
{
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    
    [BindNever]
    public bool IsAdmin { get; set; }
    
    [BindNever]
    public DateTime CreatedAt { get; set; }
}
```

### The Better Solution: Separate View Models

The most robust defense against over-posting is to use separate models for input (what the client can send) and domain (what the system manages):

```csharp
// Only contains properties the user is allowed to set
public class UpdateProfileRequest
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; } = string.Empty;
    
    [Required]
    [EmailAddress]
    public string Email { get; set; } = string.Empty;
}

[HttpPost("profile")]
public IActionResult UpdateProfile(UpdateProfileRequest request)
{
    // No risk of over-posting -- the model simply does not have
    // IsAdmin, CreatedAt, or any other sensitive properties
    
    var profile = _repository.GetCurrentUser();
    profile.Name = request.Name;
    profile.Email = request.Email;
    _repository.Update(profile);
    
    return Ok();
}
```

```ad-tip
Prefer using dedicated input models (DTOs/view models) over `[Bind]` or `[BindNever]`. Dedicated models make the contract between client and server explicit, are easier to validate, and cannot accidentally expose new properties when the domain model changes.
```

---

## Real-World Example

This example demonstrates a complete order creation flow that combines multiple binding sources: route values, form data, file uploads, and DI services.

### The Model Classes

```csharp
public class CreateOrderRequest
{
    [Required]
    public int CustomerId { get; set; }
    
    [Required]
    [StringLength(500)]
    public string? Notes { get; set; }
    
    [Required]
    [MinLength(1, ErrorMessage = "At least one item is required.")]
    public List<OrderItemInput> Items { get; set; } = new();
    
    public AddressInput ShippingAddress { get; set; } = new();
}

public class OrderItemInput
{
    [Required]
    public int ProductId { get; set; }
    
    [Range(1, 1000)]
    public int Quantity { get; set; }
}

public class AddressInput
{
    [Required]
    [StringLength(200)]
    public string Street { get; set; } = string.Empty;
    
    [Required]
    [StringLength(100)]
    public string City { get; set; } = string.Empty;
    
    [Required]
    [StringLength(50)]
    public string State { get; set; } = string.Empty;
    
    [Required]
    [RegularExpression(@"^\d{5}(-\d{4})?$")]
    public string ZipCode { get; set; } = string.Empty;
}
```

### The Controller Action

```csharp
[ApiController]
[Route("api/stores/{storeId}/orders")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;
    
    public OrdersController(IOrderService orderService)
    {
        _orderService = orderService;
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateOrder(
        [FromRoute] int storeId,                     // From URL: /api/stores/7/orders
        [FromBody] CreateOrderRequest request,       // From JSON body
        [FromHeader(Name = "X-Idempotency-Key")] string? idempotencyKey,  // From header
        [FromServices] ILogger<OrdersController> logger)  // From DI
    {
        logger.LogInformation(
            "Creating order for store {StoreId}, customer {CustomerId}, idempotency key: {Key}",
            storeId, request.CustomerId, idempotencyKey);
        
        var order = await _orderService.CreateAsync(storeId, request, idempotencyKey);
        
        return CreatedAtAction(
            nameof(GetOrder),
            new { storeId, orderId = order.Id },
            order);
    }
    
    [HttpGet("{orderId}")]
    public async Task<IActionResult> GetOrder(
        [FromRoute] int storeId,
        [FromRoute] int orderId)
    {
        var order = await _orderService.GetAsync(storeId, orderId);
        return order is null ? NotFound() : Ok(order);
    }
}
```

### Example JSON Request

```json
POST /api/stores/7/orders HTTP/1.1
Content-Type: application/json
X-Idempotency-Key: ord-2024-abc123

{
    "customerId": 42,
    "notes": "Please gift wrap the items",
    "items": [
        { "productId": 101, "quantity": 2 },
        { "productId": 205, "quantity": 1 },
        { "productId": 310, "quantity": 5 }
    ],
    "shippingAddress": {
        "street": "742 Evergreen Terrace",
        "city": "Springfield",
        "state": "IL",
        "zipCode": "62704"
    }
}
```

### Form-Based Order with File Attachment

For a form-based version that includes a file attachment (e.g., a purchase order document):

```csharp
[HttpPost("with-attachment")]
public async Task<IActionResult> CreateOrderWithAttachment(
    [FromRoute] int storeId,
    [FromForm] CreateOrderRequest request,
    [FromForm] IFormFile? purchaseOrderDocument)
{
    if (purchaseOrderDocument is not null)
    {
        if (purchaseOrderDocument.Length > 10 * 1024 * 1024)
            return BadRequest("Purchase order document must be under 10 MB.");
        
        if (purchaseOrderDocument.ContentType != "application/pdf")
            return BadRequest("Only PDF documents are accepted.");
    }
    
    var order = await _orderService.CreateAsync(storeId, request);
    
    if (purchaseOrderDocument is not null)
    {
        using var stream = purchaseOrderDocument.OpenReadStream();
        await _orderService.AttachDocumentAsync(order.Id, stream, purchaseOrderDocument.FileName);
    }
    
    return CreatedAtAction(nameof(GetOrder), new { storeId, orderId = order.Id }, order);
}
```

The corresponding HTML form:

```html
<form method="post"
      action="/api/stores/7/orders/with-attachment"
      enctype="multipart/form-data">
    
    <input type="hidden" name="CustomerId" value="42" />
    <textarea name="Notes" placeholder="Order notes..."></textarea>
    
    <div>
        <input type="number" name="Items[0].ProductId" value="101" />
        <input type="number" name="Items[0].Quantity" value="2" />
    </div>
    <div>
        <input type="number" name="Items[1].ProductId" value="205" />
        <input type="number" name="Items[1].Quantity" value="1" />
    </div>
    
    <input type="text" name="ShippingAddress.Street" placeholder="Street" />
    <input type="text" name="ShippingAddress.City" placeholder="City" />
    <input type="text" name="ShippingAddress.State" placeholder="State" />
    <input type="text" name="ShippingAddress.ZipCode" placeholder="ZIP Code" />
    
    <input type="file" name="PurchaseOrderDocument" accept=".pdf" />
    
    <button type="submit">Place Order</button>
</form>
```

---

## Comprehensive Summary

```ad-summary
**Model binding** automatically maps HTTP request data to action method parameters and model properties, eliminating manual extraction from `Request.Query`, `Request.Form`, and `Request.RouteValues`.

**Key points:**

- The default binding source priority is: Form values, Route values, Query string. Headers and Body require explicit attributes.
- Explicit source attributes (`[FromBody]`, `[FromRoute]`, `[FromQuery]`, `[FromHeader]`, `[FromForm]`, `[FromServices]`) override default priority and should be used for clarity.
- Simple types are converted from strings using `TypeConverter`. Conversion failures populate `ModelState` errors.
- Complex types are recursively bound property-by-property, with support for nested objects via dot notation.
- `[ApiController]` changes inference: complex types default to `[FromBody]`, simple types to `[FromRoute]` or `[FromQuery]`.
- Collections (arrays, lists, dictionaries) bind from repeated query parameters or indexed form fields.
- File uploads use `IFormFile` / `IFormFileCollection` with `multipart/form-data` encoding.
- Custom model binders implement `IModelBinder` for scenarios the built-in binders do not cover.
- `[BindProperty]` opts controller/PageModel properties into binding; `SupportsGet = true` enables GET binding.
- Protect against over-posting with `[Bind]`, `[BindNever]`, or (preferred) dedicated input DTOs.
```

---

## Related Topics

- [[Controllers Overview]] -- controller fundamentals and conventions
- [[Action Results]] -- returning responses from actions
- [[Validation]] -- validating bound models with data annotations and custom validators
- [[Filters]] -- action filters that run before/after model binding
- [[17.05 - Routing]] -- how requests are matched to actions before binding occurs
