---
tags:
  - csharp
  - asp-net-core
  - controllers
  - mvc
---


Every controller has access to the full HTTP context through inherited properties.

### HttpContext

The `HttpContext` object is the container for everything related to the current HTTP request and response. It is available as `this.HttpContext` inside any controller.

### Request

The `Request` property (`HttpRequest`) gives access to all incoming data:

```csharp
[HttpGet]
public IActionResult Inspect()
{
    var method = Request.Method;               // "GET"
    var path = Request.Path;                   // "/api/products"
    var queryString = Request.QueryString;     // "?page=2&size=10"
    var host = Request.Host;                   // "localhost:5001"
    var scheme = Request.Scheme;               // "https"
    var contentType = Request.ContentType;     // "application/json"
    var isHttps = Request.IsHttps;             // true

    // Query parameters
    var page = Request.Query["page"];          // "2"

    // Headers
    var userAgent = Request.Headers["User-Agent"];
    var correlationId = Request.Headers["X-Correlation-Id"];

    return Ok(new { method, path, host, correlationId = correlationId.ToString() });
}
```

### Response

The `Response` property (`HttpResponse`) lets you modify the outgoing response:

```csharp
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id)
{
    var product = await _productRepo.GetByIdAsync(id);
    if (product is null)
        return NotFound();

    // Add custom response headers
    Response.Headers.Append("X-Product-Version", product.Version.ToString());
    Response.Headers.Append("Cache-Control", "public, max-age=60");

    return Ok(product);
}
```

### User

The `User` property is a `ClaimsPrincipal` representing the authenticated user:

```csharp
[Authorize]
[HttpGet("profile")]
public IActionResult GetProfile()
{
    var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
    var email = User.FindFirstValue(ClaimTypes.Email);
    var roles = User.FindAll(ClaimTypes.Role).Select(c => c.Value);
    var isAdmin = User.IsInRole("Admin");

    return Ok(new { userId, email, roles, isAdmin });
}
```

### Reading a Custom Header -- Practical Example

```csharp
[HttpPost]
public async Task<IActionResult> ProcessWebhook()
{
    // Verify the webhook signature from a custom header
    if (!Request.Headers.TryGetValue("X-Webhook-Signature", out var signature))
    {
        return BadRequest("Missing webhook signature header");
    }

    using var reader = new StreamReader(Request.Body);
    var body = await reader.ReadToEndAsync();

    if (!_webhookService.VerifySignature(body, signature!))
    {
        return Unauthorized("Invalid webhook signature");
    }

    await _webhookService.ProcessAsync(body);
    return Ok();
}
```
