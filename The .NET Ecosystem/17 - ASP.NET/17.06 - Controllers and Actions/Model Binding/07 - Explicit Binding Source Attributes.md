---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


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
