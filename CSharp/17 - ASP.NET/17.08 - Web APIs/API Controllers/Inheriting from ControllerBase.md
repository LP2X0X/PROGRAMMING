---
tags:
  - csharp
  - asp-net-core
  - web-api
  - controllers
---


API controllers should inherit from **`ControllerBase`**, not from `Controller`.

```
System.Object
  -> ControllerBase      <-- Use this for APIs
      -> Controller      <-- Use this for MVC (adds View support)
```

**`ControllerBase`** provides everything an API controller needs:

- `Ok()`, `Ok(value)` -- 200 OK
- `Created()`, `CreatedAtAction()`, `CreatedAtRoute()` -- 201 Created
- `NoContent()` -- 204 No Content
- `BadRequest()`, `BadRequest(error)` -- 400 Bad Request
- `Unauthorized()` -- 401 Unauthorized
- `Forbid()` -- 403 Forbidden
- `NotFound()`, `NotFound(value)` -- 404 Not Found
- `Conflict()`, `Conflict(error)` -- 409 Conflict
- `UnprocessableEntity()` -- 422 Unprocessable Entity
- `StatusCode(int)` -- Any status code
- `File()` -- File responses
- `HttpContext`, `Request`, `Response` -- Access to the request pipeline
- `ModelState` -- Model validation state
- `User` -- The authenticated user's claims

**`Controller`** adds view-related members like `View()`, `PartialView()`, `ViewData`, `ViewBag`, and `TempData`. Since API controllers never render views, using `Controller` adds unnecessary overhead and can be misleading.

> [!danger]
> Inheriting from `Controller` instead of `ControllerBase` for an API controller is a common beginner mistake. It works, but it pollutes your controller with view-related methods and properties that have no purpose in an API context. Always use `ControllerBase` for API controllers.

```csharp
// CORRECT for API controllers
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    // API actions here
}

// INCORRECT for API controllers -- unnecessary view support
[ApiController]
[Route("api/[controller]")]
public class OrdersController : Controller  // Don't do this
{
    // View(), ViewBag, TempData are all available but useless
}
```

> [!summary] Section Summary
> Always inherit from `ControllerBase` for API controllers, not `Controller`. `ControllerBase` provides all the HTTP response helper methods (`Ok()`, `NotFound()`, `BadRequest()`, etc.) without the unnecessary view-rendering baggage that `Controller` adds.
