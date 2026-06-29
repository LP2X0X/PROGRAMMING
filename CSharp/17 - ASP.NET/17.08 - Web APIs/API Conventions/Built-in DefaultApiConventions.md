---
tags:
  - csharp
  - asp-net-core
  - web-api
  - conventions
  - openapi
---


ASP.NET Core ships with **`DefaultApiConventions`**, a static class that defines standard response type conventions for typical CRUD operations. Understanding what it covers helps you know when you can rely on it and when you need custom conventions.

### What DefaultApiConventions Defines

| Convention Method | Matched Action Names | Produces |
|---|---|---|
| `Get(id)` | `Get`, `Find`, `GetById` | 200, 404, default |
| `Post(model)` | `Post`, `Create` | 201, 400, default |
| `Put(id, model)` | `Put`, `Edit`, `Update` | 204, 400, 404, default |
| `Delete(id)` | `Delete`, `Remove` | 200, 400, 404, default |

### How Method Matching Works

`DefaultApiConventions` uses `[ApiConventionNameMatch]` on its methods:

1. **Method name** -- matched using `Prefix` behavior. An action named `GetProduct` matches `Get` because `GetProduct` starts with `Get`.
2. **Parameters** -- matched using `Any` or `Suffix` behavior depending on the convention method.

### Example: DefaultApiConventions in Action

```csharp
[assembly: ApiConventionType(typeof(DefaultApiConventions))]

namespace MyApi.Controllers;

[ApiController]
[Route("api/[controller]")]
public class CategoriesController : ControllerBase
{
    // Matches DefaultApiConventions.Get -- produces 200, 404
    [HttpGet("{id}")]
    public async Task<ActionResult<CategoryDto>> GetCategory(int id) { /* ... */ }

    // Matches DefaultApiConventions.Post -- produces 201, 400
    [HttpPost]
    public async Task<ActionResult<CategoryDto>> CreateCategory(
        CreateCategoryDto dto) { /* ... */ }

    // Matches DefaultApiConventions.Put -- produces 204, 400, 404
    [HttpPut("{id}")]
    public async Task<IActionResult> EditCategory(
        int id, UpdateCategoryDto dto) { /* ... */ }

    // Matches DefaultApiConventions.Delete -- produces 200, 400, 404
    [HttpDelete("{id}")]
    public async Task<IActionResult> RemoveCategory(int id) { /* ... */ }
}
```

> [!warning]
> `DefaultApiConventions` does not cover search/filter endpoints, batch operations, or non-CRUD actions like `Activate`, `Archive`, or `Export`. For those, you must use `[ProducesResponseType]` explicitly or create custom conventions.

> [!summary] Section Summary
> `DefaultApiConventions` provides out-of-the-box response type metadata for standard CRUD operations (Get, Post, Put, Delete). It matches by method name prefix and parameter patterns. Non-standard actions require explicit annotations or custom conventions.
