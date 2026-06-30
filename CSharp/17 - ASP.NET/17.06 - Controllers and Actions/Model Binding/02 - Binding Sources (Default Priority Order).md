---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


Model binding searches for values in a specific order. When a parameter name matches a value in multiple sources, the **first source that produces a match wins**.

The default priority order for non-`[ApiController]` controllers is:

| Priority | Source | Content Type | Included by Default |
|---|---|---|---|
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
