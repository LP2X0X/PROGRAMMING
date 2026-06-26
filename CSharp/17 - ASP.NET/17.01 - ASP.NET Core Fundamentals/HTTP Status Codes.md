---
tags: [csharp, asp-net-core, http, status-codes, web-api, fundamentals]
---

# HTTP Status Codes

> [!ad-note] About This Note
> This note covers HTTP status codes -- the three-digit numbers that every HTTP response carries to tell the client what happened. You will learn all five classes (1xx through 5xx), the most common codes in each, how they map to ASP.NET Core helper methods, and the mistakes that trip up developers transitioning from desktop to web.

---

## Table of Contents

- [[#What Are HTTP Status Codes]]
- [[#Desktop Developer Mental Model]]
- [[#The Five Classes]]
	- [[#1xx -- Informational]]
	- [[#2xx -- Success]]
	- [[#3xx -- Redirection]]
	- [[#4xx -- Client Error]]
	- [[#5xx -- Server Error]]
- [[#Quick Reference Table]]
- [[#ASP.NET Core and Status Codes]]
	- [[#Action Results and Helper Methods]]
	- [[#Problem Details for Error Responses]]
	- [[#Exception Handling Middleware]]
	- [[#Middleware Short-Circuiting]]
- [[#Common Mistakes]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## What Are HTTP Status Codes

Every HTTP response -- ==every single one== -- includes a three-digit **status code** in its first line. This code tells the client (browser, mobile app, API consumer) what the server did with the request:

```
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 1, "name": "Widget"}
```

The `200` is the status code. The `OK` is the **reason phrase** -- a human-readable label that accompanies the code. The code is what matters to software; the reason phrase is informational.

Status codes are defined in [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110) (formerly RFC 7231) and are split into five classes based on their first digit:

| First Digit | Class | Meaning |
|---|---|---|
| **1xx** | Informational | Request received, processing continues |
| **2xx** | Success | Request was successfully received, understood, and accepted |
| **3xx** | Redirection | Further action is needed to complete the request |
| **4xx** | Client Error | The request contains bad syntax or cannot be fulfilled |
| **5xx** | Server Error | The server failed to fulfill a valid request |

> [!ad-tip] The First Digit Is the Category
> You do not need to memorize every code. If you see a status code you have never encountered, the first digit tells you the general outcome. A `418` you have never seen? It is a 4xx, so something is wrong with the request (in this case, it is the famous joke code "I'm a teapot" -- not used in production, but it illustrates the principle).

> [!summary] Section Summary
> - Every HTTP response carries a three-digit status code
> - The first digit determines the class: 1xx informational, 2xx success, 3xx redirection, 4xx client error, 5xx server error
> - The reason phrase (e.g., "OK", "Not Found") is human-readable and informational only
> - Status codes are standardized in RFC 9110

---

## Desktop Developer Mental Model

If you are coming from WinForms or WPF, you are used to a world where methods return values or throw exceptions. In web development, the HTTP status code fills both of those roles.

Think of it this way:

| Desktop (WinForms/WPF) | Web (HTTP) | Status Code |
|---|---|---|
| Method returns a result | Response body contains data | **2xx** (Success) |
| Method returns `null` / item not found | Resource does not exist | **404** Not Found |
| `ArgumentException` / validation failure | Request has bad data | **400** Bad Request |
| `UnauthorizedAccessException` | Not logged in | **401** Unauthorized |
| `SecurityException` / no permission | Logged in but forbidden | **403** Forbidden |
| Unhandled exception crashes the app | Server error | **500** Internal Server Error |
| `Response.Redirect()` in code | Follow a different URL | **3xx** (Redirection) |

> [!ad-note] The Status Code IS the Return Value's Category
> In desktop code, you might write `if (result == null) ShowError()`. In web, the status code tells the caller the category of the outcome *before* they even look at the body. A well-behaved API consumer checks the status code first and only parses the body if the code makes sense.

```csharp
// Desktop approach (WinForms)
var product = _repository.GetById(id);
if (product == null)
    throw new NotFoundException($"Product {id} not found");
return product;

// Web approach (ASP.NET Core API Controller)
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null)
        return NotFound();  // Returns 404 status code

    return Ok(product);     // Returns 200 status code + JSON body
}
```

> [!summary] Section Summary
> - In desktop apps, you use return values and exceptions to communicate outcomes
> - In web, the status code replaces both -- it tells the caller whether to celebrate (2xx), go elsewhere (3xx), fix their request (4xx), or panic (5xx)
> - A well-designed API always uses the correct status code, not just 200 with an error message in the body
> - The status code is checked before the body is parsed

---

## The Five Classes

### 1xx -- Informational

**Informational responses** indicate that the server has received the request and is continuing to process it. You will rarely interact with these directly in typical ASP.NET Core development -- they are handled by the HTTP infrastructure (Kestrel, reverse proxies, browsers).

| Code | Name | When It Happens |
|---|---|---|
| **100** | Continue | Client sent headers with `Expect: 100-continue`. Server says "go ahead and send the body." Kestrel handles this automatically. |
| **101** | Switching Protocols | Used during [[WebSockets]] upgrade. The connection switches from HTTP to the WebSocket protocol. |
| **102** | Processing | Indicates the server is working on a long request (WebDAV). Rarely seen. |

> [!ad-note] You Almost Never Set These Yourself
> In ASP.NET Core, 1xx codes are managed by Kestrel and the HTTP stack. When a client initiates a WebSocket connection, Kestrel sends the `101 Switching Protocols` response automatically. You do not return `StatusCode(100)` from your controllers.

```csharp
// You don't manually return 1xx codes.
// WebSocket upgrade is handled by the framework:
app.UseWebSockets();
app.Map("/ws", async context =>
{
    if (context.WebSockets.IsWebSocketRequest)
    {
        // Framework sends 101 Switching Protocols automatically
        var ws = await context.WebSockets.AcceptWebSocketAsync();
        // ... handle WebSocket communication
    }
    else
    {
        context.Response.StatusCode = 400;
    }
});
```

> [!summary] Section Summary
> - 1xx codes are informational -- the server acknowledges receipt and continues processing
> - 100 Continue and 101 Switching Protocols are the two you might encounter
> - Kestrel and the HTTP infrastructure handle these automatically
> - You almost never set 1xx codes manually in your controllers

---

### 2xx -- Success

**Success responses** mean the server received the request, understood it, and processed it successfully. These are the "happy path" codes.

| Code | Name | Meaning | ASP.NET Core Helper |
|---|---|---|---|
| **200** | OK | Standard success response. Body contains the requested data. | `Ok()` / `Ok(value)` |
| **201** | Created | A new resource was successfully created. Should include a `Location` header pointing to the new resource. | `Created()` / `CreatedAtAction()` / `CreatedAtRoute()` |
| **202** | Accepted | Request was accepted for processing, but processing is not complete. Used for async/queued operations. | `Accepted()` |
| **204** | No Content | Success, but there is no body to return. Common for DELETE and PUT operations. | `NoContent()` |

#### 200 OK -- The Default Success

The most common status code. The request succeeded and the response body contains data.

```csharp
[HttpGet]
public async Task<IActionResult> GetAll()
{
    var products = await _repository.GetAllAsync();
    return Ok(products); // 200 + JSON array
}
```

#### 201 Created -- Resource Was Created

Used after a successful POST that creates a new resource. Best practice is to include a `Location` header that points to the newly created resource's URL.

```csharp
[HttpPost]
public async Task<IActionResult> Create(CreateProductDto dto)
{
    var product = await _service.CreateAsync(dto);

    // Returns 201 with Location header pointing to GetById endpoint
    return CreatedAtAction(
        actionName: nameof(GetById),
        routeValues: new { id = product.Id },
        value: product
    );
}
```

The response will look like:

```
HTTP/1.1 201 Created
Location: /api/products/42
Content-Type: application/json

{"id": 42, "name": "New Widget", "price": 19.99}
```

#### 204 No Content -- Success, Nothing to Return

Used when the operation succeeds but there is nothing meaningful to return in the body. Common for `DELETE` and `PUT` operations.

```csharp
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(int id)
{
    var existed = await _repository.DeleteAsync(id);
    if (!existed)
        return NotFound();

    return NoContent(); // 204 -- deleted successfully, nothing to return
}

[HttpPut("{id}")]
public async Task<IActionResult> Update(int id, UpdateProductDto dto)
{
    var product = await _repository.GetByIdAsync(id);
    if (product is null)
        return NotFound();

    await _service.UpdateAsync(id, dto);
    return NoContent(); // 204 -- updated successfully
}
```

> [!ad-warning] Common Misconception: Always Return 200
> Beginners often return `200 OK` for everything -- including creates and deletes. This makes the API less self-describing. Use `201 Created` when something was created and `204 No Content` when there is nothing to return. Your API consumers (and tools like Swagger) benefit from the precision.

> [!ad-tip] 202 Accepted -- For Async Operations
> If your API kicks off a long-running background job (e.g., generating a report, processing an upload), return `202 Accepted` to indicate "I received your request and will process it, but it is not done yet." Optionally include a URL where the client can poll for status.
> ```csharp
> [HttpPost("reports")]
> public IActionResult GenerateReport(ReportRequest request)
> {
>     var jobId = _backgroundJobService.Enqueue(request);
>     return Accepted(
>         uri: $"/api/reports/status/{jobId}",
>         value: new { jobId, status = "processing" }
>     );
> }
> ```

> [!summary] Section Summary
> - 200 OK: standard success with data in the body
> - 201 Created: resource was created; include a `Location` header pointing to it
> - 202 Accepted: request accepted for async processing; not yet complete
> - 204 No Content: success but no body to return (DELETE, PUT)
> - Use the precise code, not 200 for everything -- it makes your API self-describing

---

### 3xx -- Redirection

**Redirection responses** tell the client that it needs to take additional action -- usually following a different URL -- to complete the request. These are most relevant in MVC/Razor Pages applications where the browser follows redirects automatically.

| Code | Name | Meaning | ASP.NET Core Helper |
|---|---|---|---|
| **301** | Moved Permanently | The resource has permanently moved to a new URL. Browsers and search engines update their bookmarks/indexes. | `RedirectPermanent()` / `RedirectToActionPermanent()` |
| **302** | Found | Temporary redirect. The resource is temporarily at a different URL. Browser follows but does not update bookmarks. | `Redirect()` / `RedirectToAction()` |
| **303** | See Other | Redirect after a POST (Post-Redirect-Get pattern). Browser should use GET for the redirect. | `RedirectToAction()` (after POST) |
| **304** | Not Modified | The resource has not changed since the client's last request. Browser should use its cached copy. | Handled automatically by response caching middleware |
| **307** | Temporary Redirect | Like 302, but the browser must use the same HTTP method (e.g., POST stays POST). | `RedirectPreserveMethod()` |
| **308** | Permanent Redirect | Like 301, but the browser must use the same HTTP method. | `RedirectPermanentPreserveMethod()` |

#### 301 vs 302 -- Permanent vs Temporary

```csharp
// 301 -- URL has permanently changed (search engines will update their index)
[HttpGet("old-products")]
public IActionResult OldProductsPage()
{
    return RedirectPermanent("/products"); // 301
}

// 302 -- Temporary redirect (search engines keep the old URL)
[HttpGet("promo")]
public IActionResult CurrentPromo()
{
    return Redirect("/products/summer-sale-2026"); // 302
}
```

#### The Post-Redirect-Get Pattern

One of the most common patterns in MVC/Razor Pages. After processing a form submission (POST), redirect the user to a GET page to prevent duplicate form submissions when the user refreshes.

```csharp
[HttpPost]
public async Task<IActionResult> Create(CreateProductViewModel model)
{
    if (!ModelState.IsValid)
        return View(model); // Return 200 with validation errors

    await _service.CreateAsync(model);

    // 302 redirect to the Index page (GET)
    // Prevents duplicate submission on browser refresh
    return RedirectToAction(nameof(Index));
}
```

#### 304 Not Modified -- Caching

When a browser has a cached copy of a resource, it can send a conditional request with `If-None-Match` (ETag) or `If-Modified-Since` headers. If the resource has not changed, the server returns `304 Not Modified` with no body -- the browser uses its cache.

```csharp
// In Program.cs -- enable response caching middleware
app.UseResponseCaching();

// On a controller action
[HttpGet("{id}")]
[ResponseCache(Duration = 60)] // Cache for 60 seconds
public async Task<IActionResult> GetById(int id)
{
    var product = await _repository.GetByIdAsync(id);
    return product is null ? NotFound() : Ok(product);
}
```

> [!ad-note] APIs vs Browser Redirects
> In REST APIs, 3xx codes are less common because API clients (e.g., `HttpClient`) typically handle redirects programmatically or expect the correct URL upfront. In browser-based MVC/Razor Pages apps, redirects are a fundamental navigation mechanism.

> [!summary] Section Summary
> - 301/308: permanent redirect -- browsers and search engines update their stored URLs
> - 302/307: temporary redirect -- browsers follow but keep the original URL
> - 303: used after POST to redirect to a GET page (Post-Redirect-Get pattern)
> - 304: resource unchanged, browser should use its cached copy
> - Redirects are common in MVC/Razor Pages; less common in REST APIs

---

### 4xx -- Client Error

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

---

### 5xx -- Server Error

**Server error responses** mean ==the problem is on the server's side==. The client's request was valid, but the server failed to process it. These indicate bugs, infrastructure issues, or temporary failures.

| Code | Name | Meaning |
|---|---|---|
| **500** | Internal Server Error | The server encountered an unexpected condition. The catch-all "something broke" code. |
| **502** | Bad Gateway | The server, acting as a gateway or proxy, received an invalid response from an upstream server. |
| **503** | Service Unavailable | The server is temporarily unable to handle requests (overloaded or down for maintenance). |
| **504** | Gateway Timeout | The server, acting as a gateway or proxy, did not receive a timely response from an upstream server. |

#### 500 Internal Server Error -- Something Broke

This is the "unhandled exception" of HTTP. In ASP.NET Core, unhandled exceptions are caught by the [[Exception Handling]] middleware and converted to 500 responses.

```csharp
// You should NEVER return 500 manually in normal flow.
// The exception handling middleware does this for you:

// In Program.cs
app.UseExceptionHandler("/error"); // Catches unhandled exceptions → 500

// Or with Problem Details (recommended):
app.UseExceptionHandler();
builder.Services.AddProblemDetails();
```

> [!ad-warning] Never Return 500 for Validation Errors
> A 500 means "the server has a bug." If the client sent bad data, that is a **400** (or 422), not a 500. Returning 500 for validation errors tells the client "this is our fault" when it is actually theirs.
> ```csharp
> // WRONG -- 500 for bad input
> [HttpPost]
> public IActionResult Create(CreateProductDto dto)
> {
>     if (string.IsNullOrEmpty(dto.Name))
>         throw new Exception("Name is required"); // Becomes 500!
>
>     // ...
> }
>
> // CORRECT -- 400 for bad input
> [HttpPost]
> public IActionResult Create(CreateProductDto dto)
> {
>     if (string.IsNullOrEmpty(dto.Name))
>         return BadRequest(new { error = "Name is required" }); // 400
>
>     // ...
> }
> ```

#### 502, 503, 504 -- Infrastructure Codes

These are typically not returned by your ASP.NET Core application code directly. They come from reverse proxies (Nginx, IIS, Azure App Gateway, load balancers) or from your code when it acts as a proxy to another service.

```
Client → Nginx (reverse proxy) → Kestrel (your app) → Database / External API
                 │                       │
                 │                       └── If your app crashes → Nginx returns 502
                 │
                 └── If Kestrel doesn't respond in time → Nginx returns 504
```

| Scenario | Who Returns the Code |
|---|---|
| Your app throws an unhandled exception | Your app returns **500** |
| Your app crashes or Kestrel goes down | Reverse proxy (Nginx/IIS) returns **502** |
| Your app is overloaded or starting up | Reverse proxy returns **503** |
| Your app takes too long to respond | Reverse proxy returns **504** |
| Your app calls an external API that fails | Your app might return **502** (acting as a gateway) |

```csharp
// When your app acts as a gateway to another service:
[HttpGet("external-data")]
public async Task<IActionResult> GetExternalData()
{
    try
    {
        var response = await _httpClient.GetAsync("https://api.thirdparty.com/data");

        if (!response.IsSuccessStatusCode)
            return StatusCode(502, new { error = "Upstream service returned an error" });

        var data = await response.Content.ReadFromJsonAsync<ExternalData>();
        return Ok(data);
    }
    catch (TaskCanceledException) // Timeout
    {
        return StatusCode(504, new { error = "Upstream service timed out" });
    }
    catch (HttpRequestException)
    {
        return StatusCode(502, new { error = "Could not reach upstream service" });
    }
}
```

> [!summary] Section Summary
> - 5xx codes mean the server failed to handle a valid request
> - 500: catch-all server error -- unhandled exceptions become 500 via exception handling middleware
> - 502/503/504: infrastructure codes typically returned by reverse proxies, not your application code
> - Never manually return 500 for validation errors -- that is a 400 or 422
> - When your app acts as a gateway to other services, 502 and 504 become relevant

---

## Quick Reference Table

A consolidated reference of every common HTTP status code, its meaning, and the ASP.NET Core helper method (if applicable).

| Code | Name | Category | Meaning | ASP.NET Core Helper |
|---|---|---|---|---|
| 100 | Continue | Informational | Send the body | *(automatic)* |
| 101 | Switching Protocols | Informational | Upgrading to WebSocket | *(automatic)* |
| **200** | OK | Success | Standard success | `Ok()` / `Ok(value)` |
| **201** | Created | Success | Resource created | `CreatedAtAction()` / `CreatedAtRoute()` |
| **202** | Accepted | Success | Queued for processing | `Accepted()` |
| **204** | No Content | Success | Success, no body | `NoContent()` |
| **301** | Moved Permanently | Redirect | URL permanently changed | `RedirectPermanent()` |
| **302** | Found | Redirect | Temporary redirect | `Redirect()` / `RedirectToAction()` |
| **304** | Not Modified | Redirect | Use cached version | *(response caching middleware)* |
| **307** | Temporary Redirect | Redirect | Temp redirect, keep method | `RedirectPreserveMethod()` |
| **400** | Bad Request | Client Error | Malformed / invalid data | `BadRequest()` |
| **401** | Unauthorized | Client Error | Not authenticated | `Unauthorized()` / `Challenge()` |
| **403** | Forbidden | Client Error | Authenticated, no permission | `Forbid()` |
| **404** | Not Found | Client Error | Resource does not exist | `NotFound()` |
| **405** | Method Not Allowed | Client Error | Wrong HTTP method | *(automatic by routing)* |
| **409** | Conflict | Client Error | State conflict | `Conflict()` |
| **415** | Unsupported Media Type | Client Error | Wrong Content-Type | *(automatic by `[ApiController]`)* |
| **422** | Unprocessable Entity | Client Error | Semantically invalid | `UnprocessableEntity()` |
| **429** | Too Many Requests | Client Error | Rate limited | *(rate limiting middleware)* |
| **500** | Internal Server Error | Server Error | Unhandled exception | `StatusCode(500)` *(via middleware)* |
| **502** | Bad Gateway | Server Error | Upstream error | `StatusCode(502)` |
| **503** | Service Unavailable | Server Error | Server overloaded/down | `StatusCode(503)` |
| **504** | Gateway Timeout | Server Error | Upstream timeout | `StatusCode(504)` |

> [!ad-tip] Using `StatusCode()` for Any Code
> For status codes that do not have a dedicated helper method, use the generic `StatusCode()` method:
> ```csharp
> return StatusCode(418); // I'm a teapot
> return StatusCode(429, new { error = "Rate limit exceeded", retryAfter = 60 });
> return StatusCode(503, new { error = "Service temporarily unavailable" });
> ```

> [!summary] Section Summary
> - Most common codes have dedicated ASP.NET Core helper methods (`Ok()`, `NotFound()`, `BadRequest()`, etc.)
> - Some codes are returned automatically by the framework (`405`, `415`, `304`)
> - Use `StatusCode(int)` for any code that lacks a dedicated helper
> - The table above covers the codes you will encounter in 99% of web development

---

## ASP.NET Core and Status Codes

### Action Results and Helper Methods

In ASP.NET Core, controllers return **action results** that encapsulate the status code and response body. The `ControllerBase` class provides helper methods that create these results.

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;

    public OrdersController(IOrderService orderService)
    {
        _orderService = orderService;
    }

    // GET /api/orders → 200 OK
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        var orders = await _orderService.GetAllAsync();
        return Ok(orders);
    }

    // GET /api/orders/5 → 200 OK or 404 Not Found
    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id)
    {
        var order = await _orderService.GetByIdAsync(id);
        return order is null ? NotFound() : Ok(order);
    }

    // POST /api/orders → 201 Created
    [HttpPost]
    public async Task<IActionResult> Create(CreateOrderDto dto)
    {
        var order = await _orderService.CreateAsync(dto);
        return CreatedAtAction(nameof(GetById), new { id = order.Id }, order);
    }

    // PUT /api/orders/5 → 204 No Content or 404 Not Found
    [HttpPut("{id}")]
    public async Task<IActionResult> Update(int id, UpdateOrderDto dto)
    {
        var order = await _orderService.GetByIdAsync(id);
        if (order is null)
            return NotFound();

        await _orderService.UpdateAsync(id, dto);
        return NoContent();
    }

    // DELETE /api/orders/5 → 204 No Content or 404 Not Found
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var existed = await _orderService.DeleteAsync(id);
        return existed ? NoContent() : NotFound();
    }
}
```

> [!ad-note] `IActionResult` vs `ActionResult<T>` vs Concrete Types
> There are three patterns for declaring return types:
> ```csharp
> // 1. IActionResult -- can return any status code, no type info for Swagger
> public IActionResult GetById(int id) => Ok(product);
>
> // 2. ActionResult<T> -- type info for Swagger + can return different status codes
> public ActionResult<Product> GetById(int id)
> {
>     var product = _repo.GetById(id);
>     if (product is null) return NotFound();   // 404
>     return product;                            // 200 (implicit Ok)
> }
>
> // 3. Concrete type -- always 200, no flexibility for error codes
> public Product GetById(int id) => _repo.GetById(id); // Always 200
> ```
> For APIs, `ActionResult<T>` is generally preferred because it gives Swagger the return type while still allowing you to return error codes.

#### Minimal API Equivalents

In [[Minimal APIs]], the helper methods are on the static `Results` (or `TypedResults`) class:

```csharp
var app = WebApplication.Create(args);

app.MapGet("/api/products/{id}", async (int id, IProductRepository repo) =>
{
    var product = await repo.GetByIdAsync(id);
    return product is null
        ? Results.NotFound()                    // 404
        : Results.Ok(product);                  // 200
});

app.MapPost("/api/products", async (CreateProductDto dto, IProductService service) =>
{
    var product = await service.CreateAsync(dto);
    return Results.Created($"/api/products/{product.Id}", product); // 201
});

app.MapDelete("/api/products/{id}", async (int id, IProductRepository repo) =>
{
    var existed = await repo.DeleteAsync(id);
    return existed
        ? Results.NoContent()                   // 204
        : Results.NotFound();                   // 404
});
```

---

### Problem Details for Error Responses

[[Problem Details]] (RFC 9457, formerly RFC 7807) is the standard format for returning machine-readable error information in HTTP APIs. ASP.NET Core has built-in support.

```json
{
    "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
    "title": "Bad Request",
    "status": 400,
    "detail": "The 'Price' field must be greater than 0.",
    "instance": "/api/products",
    "traceId": "00-abc123-def456-01"
}
```

Enable Problem Details globally:

```csharp
// In Program.cs
builder.Services.AddProblemDetails();

// The [ApiController] attribute automatically uses Problem Details
// for 400 validation errors and other client errors
```

> [!ad-tip] Consistent Error Responses
> With Problem Details enabled, all your 4xx and 5xx responses follow a consistent JSON structure. This makes it much easier for API consumers to handle errors programmatically -- they always know where to find the error message, status code, and trace ID.

---

### Exception Handling Middleware

The [[Exception Handling]] middleware catches unhandled exceptions and converts them to 500 responses. Without it, an unhandled exception would crash the request with no response.

```csharp
var app = builder.Build();

// Add exception handling FIRST in the pipeline
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage(); // Detailed error page (HTML)
}
else
{
    app.UseExceptionHandler(); // Returns Problem Details JSON for APIs
}
```

In development, `UseDeveloperExceptionPage()` shows a detailed HTML error page with the stack trace. In production, `UseExceptionHandler()` returns a sanitized error response (no stack traces leaked to clients).

---

### Middleware Short-Circuiting

Any [[Middleware Overview|middleware]] in the pipeline can ==short-circuit the request== and return a status code directly, without reaching your controller.

```csharp
// Custom middleware that rejects requests without an API key
app.Use(async (context, next) =>
{
    if (!context.Request.Headers.ContainsKey("X-Api-Key"))
    {
        context.Response.StatusCode = 401; // Short-circuit with 401
        await context.Response.WriteAsJsonAsync(new
        {
            error = "API key is required. Include an X-Api-Key header."
        });
        return; // Do NOT call next() -- request stops here
    }

    await next(context); // API key present, continue to next middleware
});
```

Common middleware that returns status codes:

| Middleware | Status Code | When |
|---|---|---|
| Authentication | 401 | No valid credentials |
| Authorization | 403 | Insufficient permissions |
| Rate Limiting | 429 | Too many requests |
| CORS | 403/204 | Origin not allowed / preflight |
| Static Files | 200/304/404 | File found/cached/missing |
| HTTPS Redirection | 307 | HTTP request redirected to HTTPS |

> [!summary] Section Summary
> - `ControllerBase` provides helper methods like `Ok()`, `NotFound()`, `BadRequest()` that set status codes
> - `ActionResult<T>` is preferred over `IActionResult` for APIs because it provides type information for Swagger
> - Minimal APIs use `Results.Ok()`, `Results.NotFound()`, etc. instead of controller helpers
> - [[Problem Details]] standardizes error response format for 4xx/5xx codes
> - [[Exception Handling]] middleware catches unhandled exceptions and returns 500
> - Any middleware can short-circuit the pipeline and return a status code directly

---

## Common Mistakes

### Mistake 1: Confusing 401 and 403

As covered above, 401 means "not authenticated" and 403 means "not authorized." Because 401 is named "Unauthorized," developers frequently mix them up.

```csharp
// WRONG -- returning 403 when the user is not logged in
if (User.Identity?.IsAuthenticated != true)
    return Forbid(); // 403 -- but this should be 401!

// CORRECT
if (User.Identity?.IsAuthenticated != true)
    return Unauthorized(); // 401 -- not authenticated

if (!User.IsInRole("Admin"))
    return Forbid(); // 403 -- authenticated but not authorized
```

### Mistake 2: Returning 200 with an Error Message

This is common among developers new to APIs. They return 200 OK for every response and put the error in the body.

```csharp
// WRONG -- 200 with error in body
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var product = _repo.GetById(id);
    if (product is null)
        return Ok(new { success = false, error = "Product not found" }); // 200!

    return Ok(new { success = true, data = product });
}

// CORRECT -- use proper status codes
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var product = _repo.GetById(id);
    if (product is null)
        return NotFound(); // 404 -- the right status code

    return Ok(product);    // 200
}
```

> [!ad-warning] Why This Is Harmful
> Returning 200 for errors breaks HTTP semantics. Tools like Swagger, `HttpClient`, browser DevTools, and monitoring dashboards all rely on status codes to categorize responses. If everything is 200, you cannot distinguish success from failure without parsing the body -- and monitoring tools will report 100% success even when half your requests are failing.

### Mistake 3: Returning 500 for Validation Errors

If the client sends bad data, that is the client's fault -- a 4xx code. A 500 means the server has a bug.

```csharp
// WRONG -- validation error → 500
[HttpPost]
public IActionResult Create(CreateProductDto dto)
{
    if (dto.Price < 0)
        throw new ArgumentException("Price must be positive");
        // Unhandled → exception middleware → 500 Internal Server Error

    // ...
}

// CORRECT -- validation error → 400 or 422
[HttpPost]
public IActionResult Create(CreateProductDto dto)
{
    if (dto.Price < 0)
        return BadRequest(new { error = "Price must be positive" }); // 400

    // ...
}
```

### Mistake 4: Not Returning a Location Header with 201

When you return `201 Created`, the HTTP spec says you should include a `Location` header pointing to the newly created resource. Using `CreatedAtAction()` or `CreatedAtRoute()` does this automatically.

```csharp
// WRONG -- 201 without Location header
return StatusCode(201, product);

// CORRECT -- 201 with Location header
return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
// Response: 201 Created
// Location: /api/products/42
```

### Mistake 5: Using 404 When the Collection Is Empty

An empty collection is not "not found." The resource (the collection endpoint) exists; it just has no items.

```csharp
// WRONG -- empty list → 404
[HttpGet]
public IActionResult GetAll()
{
    var products = _repo.GetAll();
    if (!products.Any())
        return NotFound(); // 404 -- but the endpoint exists!

    return Ok(products);
}

// CORRECT -- empty list → 200 with empty array
[HttpGet]
public IActionResult GetAll()
{
    var products = _repo.GetAll();
    return Ok(products); // 200 + [] -- the resource exists, it's just empty
}
```

> [!summary] Section Summary
> - 401 = not authenticated, 403 = not authorized -- the naming is misleading
> - Never return 200 with an error message in the body -- use proper status codes
> - Never return 500 for validation errors -- that is a 400 or 422
> - Use `CreatedAtAction()` with 201 to include the `Location` header automatically
> - An empty collection is 200 with an empty array, not 404

---

## Comprehensive Summary

> [!ad-tip] Complete Summary
> **HTTP status codes** are three-digit numbers in every HTTP response that tell the client what happened. They are the primary mechanism for communicating outcomes in web APIs -- replacing the return values and exceptions you are used to from desktop development.
>
> **The five classes** are determined by the first digit:
> - **1xx (Informational)**: Server acknowledgments, handled automatically by the HTTP infrastructure. You rarely interact with these.
> - **2xx (Success)**: The request succeeded. Use **200 OK** for standard responses, **201 Created** when a resource is created (with a `Location` header), **204 No Content** for successful operations with no body (DELETE, PUT), and **202 Accepted** for queued/async operations.
> - **3xx (Redirection)**: The client should follow a different URL. **301** for permanent moves, **302** for temporary redirects, **304** for cached responses. The Post-Redirect-Get pattern (POST then redirect to GET) prevents duplicate form submissions.
> - **4xx (Client Error)**: The client sent a bad request. **400** for invalid data, **401** for "not authenticated" (despite its misleading name), **403** for "authenticated but not authorized," **404** for resources that do not exist, **409** for state conflicts, **422** for semantically invalid data.
> - **5xx (Server Error)**: The server failed. **500** is the catch-all for unhandled exceptions. **502/503/504** are infrastructure codes typically from reverse proxies.
>
> **In ASP.NET Core**, status codes are set through action result helper methods: `Ok()`, `NotFound()`, `BadRequest()`, `CreatedAtAction()`, `NoContent()`, `Unauthorized()`, `Forbid()`, `Conflict()`, `UnprocessableEntity()`. The `[ApiController]` attribute enables automatic 400 responses for model validation failures. [[Problem Details]] provides a standard JSON format for error responses. The [[Exception Handling]] middleware converts unhandled exceptions to 500 responses.
>
> **The most common mistakes** are: confusing 401 (authentication) with 403 (authorization), returning 200 with error messages in the body, throwing exceptions for validation errors (which become 500s), and returning 404 for empty collections.
>
> **Desktop developer takeaway**: The status code is the HTTP equivalent of your method's return category. 2xx = success, 3xx = "go elsewhere," 4xx = "your fault," 5xx = "my fault." Always check the status code before parsing the response body.

---

## Related Topics

- [[ASP.NET Core Overview]] -- the foundational overview of the ASP.NET Core framework
- [[Action Results]] -- detailed coverage of `IActionResult`, `ActionResult<T>`, and all result types
- [[Controllers Overview]] -- how controllers work and route to action methods
- [[API Controllers]] -- the `[ApiController]` attribute and its automatic behaviors
- [[Minimal APIs]] -- the lightweight alternative to controllers for HTTP endpoints
- [[Problem Details]] -- the RFC 9457 standard format for error responses
- [[Exception Handling]] -- middleware for catching unhandled exceptions
- [[Middleware Overview]] -- how the request pipeline works and middleware ordering
- [[Request Pipeline]] -- detailed flow of HTTP requests through the pipeline
- [[Authentication Overview]] -- authentication schemes and the 401/403 relationship
- [[Authorization Policies]] -- policy-based authorization and role checks
- [[Model Binding]] -- how ASP.NET Core binds HTTP request data to action parameters
- [[Validation]] -- model validation and its relationship to 400 responses
