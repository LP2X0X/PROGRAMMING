---
tags:
  - csharp
  - asp-net-core
  - filters
  - controllers
  - cross-cutting
---


ASP.NET Core ships with many filter attributes out of the box. These cover the most common cross-cutting concerns.

### Authorization

```csharp
// Require any authenticated user
[Authorize]
public class AccountController : Controller { }

// Require specific role
[Authorize(Roles = "Admin,Manager")]
public IActionResult DeleteUser(int id) => Ok();

// Require a named policy
[Authorize(Policy = "CanEditProducts")]
public IActionResult Edit(int id) => Ok();

// Override authorization on a specific action
[Authorize]
public class AdminController : Controller
{
    [AllowAnonymous] // Anyone can access this action
    public IActionResult PublicStatus() => Ok("Running");
}
```

See [[17.09 - Authentication and Authorization]] for defining policies.

### HTTPS and Transport Security

```csharp
// Redirect HTTP requests to HTTPS
[RequireHttps]
public class SecureController : Controller { }
```

### Response Caching

```csharp
// Cache the response for 60 seconds
[ResponseCache(Duration = 60)]
public IActionResult GetProducts() => Ok(products);

// Cache with varying by query string
[ResponseCache(Duration = 120, VaryByQueryKeys = new[] { "category", "page" })]
public IActionResult Search(string category, int page) => Ok();

// No caching
[ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
public IActionResult GetSensitiveData() => Ok();
```

### Anti-Forgery (CSRF Protection)

```csharp
// Require anti-forgery token for this POST action
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult SubmitForm(FormModel model) => Ok();

// Skip anti-forgery validation (e.g., for API endpoints using JWT)
[HttpPost]
[IgnoreAntiforgeryToken]
public IActionResult ApiEndpoint([FromBody] RequestDto dto) => Ok();
```

### Content Negotiation

```csharp
// Only accept JSON requests
[Consumes("application/json")]
[HttpPost]
public IActionResult Create([FromBody] ProductDto product) => Ok();

// Declare that this action always returns JSON
[Produces("application/json")]
public IActionResult GetAll() => Ok(products);

// Enable URL-based format selection (/api/products.json, /api/products.xml)
[FormatFilter]
[Route("api/products.{format?}")]
public IActionResult Get() => Ok(products);
```
