---
tags:
  - csharp
  - asp-net-core
  - action-results
  - controllers
---


| Helper Method | Status Code | Return Type | When to Use |
|---|---|---|---|
| `Ok()` | 200 | `OkResult` | Successful action, no body |
| `Ok(value)` | 200 | `OkObjectResult` | Successful GET, return data |
| `Created(uri, value)` | 201 | `CreatedResult` | Resource created, provide location URI |
| `CreatedAtAction(action, routeValues, value)` | 201 | `CreatedAtActionResult` | Resource created, location header points to a GET action |
| `CreatedAtRoute(routeName, routeValues, value)` | 201 | `CreatedAtRouteResult` | Resource created, location header uses a named route |
| `Accepted()` | 202 | `AcceptedResult` | Long-running operation accepted for processing |
| `NoContent()` | 204 | `NoContentResult` | Successful PUT/DELETE with no response body |
| `BadRequest()` | 400 | `BadRequestResult` | Validation failure, no details |
| `BadRequest(modelState)` | 400 | `BadRequestObjectResult` | Validation failure with error details |
| `Unauthorized()` | 401 | `UnauthorizedResult` | Client is not authenticated |
| `Forbid()` | 403 | `ForbidResult` | Authenticated but lacks permission |
| `NotFound()` | 404 | `NotFoundResult` | Resource does not exist, no details |
| `NotFound(value)` | 404 | `NotFoundObjectResult` | Resource does not exist, with error info |
| `Conflict()` | 409 | `ConflictResult` | Concurrency or state conflict, no details |
| `Conflict(value)` | 409 | `ConflictObjectResult` | Concurrency conflict with error info |
| `UnprocessableEntity()` | 422 | `UnprocessableEntityObjectResult` | Semantic validation failure |
| `StatusCode(code)` | any | `StatusCodeResult` | Custom or uncommon status codes |
| `StatusCode(code, value)` | any | `ObjectResult` | Custom status code with body |
| `View()` / `View(model)` | 200 | `ViewResult` | Render a Razor view (MVC only) |
| `PartialView()` | 200 | `PartialViewResult` | Render a partial Razor view |
| `RedirectToAction(action)` | 302 | `RedirectToActionResult` | Redirect to another action method |
| `RedirectToPage(pageName)` | 302 | `RedirectToPageResult` | Redirect to a Razor Page |
| `Redirect(url)` | 302 | `RedirectResult` | Redirect to an external or absolute URL |
| `RedirectPermanent(url)` | 301 | `RedirectResult` | Permanent redirect (SEO) |
| `Json(value)` | 200 | `JsonResult` | Force JSON serialization explicitly |
| `File(bytes, contentType)` | 200 | `FileContentResult` | File download from byte array |
| `File(stream, contentType)` | 200 | `FileStreamResult` | File download from stream |
| `PhysicalFile(path, contentType)` | 200 | `PhysicalFileResult` | File download from disk path |
| `Content(text)` | 200 | `ContentResult` | Plain text response |

```ad-info
`Forbid()` and `Unauthorized()` differ in an important way. `Unauthorized()` returns a raw 401 and does nothing else. `Forbid()` triggers the configured authentication handler's `ForbidAsync` method, which can redirect to an access-denied page. For API controllers, `Forbid()` also produces a 403, but through the authentication middleware.
```

```ad-warning
`Json()` is rarely needed when using `[ApiController]`. The framework already serializes objects as JSON via content negotiation. Use `Json()` only when you need to override serializer settings for a specific response or when returning JSON from an MVC controller that normally returns views.
```
