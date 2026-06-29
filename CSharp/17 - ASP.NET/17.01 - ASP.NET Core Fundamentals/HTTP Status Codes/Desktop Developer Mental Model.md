---
tags: [csharp, asp-net-core, http, status-codes, web-api, fundamentals]
---


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
