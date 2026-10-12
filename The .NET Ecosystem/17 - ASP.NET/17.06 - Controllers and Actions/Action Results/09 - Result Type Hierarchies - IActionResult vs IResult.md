---
tags:
  - csharp
  - asp-net-core
  - action-results
  - controllers
  - minimal-apis
---


ASP.NET Core has **two separate result type hierarchies** that serve the same purpose -- writing an HTTP response -- but target different programming models. Understanding why both exist and how they differ is essential to working fluently across controllers and [[Minimal APIs]].


## Why Two Hierarchies Exist

The split comes down to **execution context**. Each interface defines a single method, but the parameter type differs:

```csharp
public interface IActionResult
{
    Task ExecuteResultAsync(ActionContext context);
}

public interface IResult
{
    Task ExecuteAsync(HttpContext context);
}
```

`IActionResult` receives an `ActionContext`. `IResult` receives a raw `HttpContext`. That one parameter difference is the entire reason two hierarchies exist.

### The context nesting

```
HttpContext                          ← raw HTTP: request, response, connection, user
└── ActionContext                    ← MVC adds: controller descriptor, action descriptor,
                                       model state, route data, action filters
```

`ActionContext` **wraps** `HttpContext` -- it holds a reference to it via `ActionContext.HttpContext`. So any `IActionResult` can reach the raw HTTP layer when it needs to. But the reverse is not true: an `IResult` executing against a bare `HttpContext` has no access to MVC concepts like model state, action descriptors, or view engines.

This means:

- `IActionResult` implementations **can** do everything `IResult` implementations can (write status codes, headers, body), **plus** MVC-specific work (render Razor views, access `ModelStateDictionary`, read action filter context).
- `IResult` implementations **cannot** render views or access MVC metadata. They operate at the raw HTTP level.

```ad-info
Think of it this way: `IActionResult` is the MVC-aware result. `IResult` is the pipeline-minimal result. Controllers need the MVC context. [[Minimal APIs]] don't have one -- they sit directly on the endpoint routing layer with no controller, no action descriptor, and no model state dictionary.
```


## The IActionResult Hierarchy (Controllers + Razor Pages)

This is the hierarchy used by [[04 - Controller vs ControllerBase|ControllerBase]] and `Controller`. Every helper method like `Ok()`, `NotFound()`, and `View()` returns an implementation of `IActionResult`.

### Class tree

```
IActionResult
├── ActionResult                        (abstract base class)
│
├── StatusCodeResult                    (status code only, no body)
│   ├── OkResult                        // 200, no body
│   ├── NoContentResult                 // 204
│   ├── NotFoundResult                  // 404
│   ├── UnauthorizedResult              // 401
│   ├── ConflictResult                  // 409
│   └── UnprocessableEntityResult       // 422
│
├── ObjectResult                        (status code + serialized body)
│   ├── OkObjectResult                  // 200 + data
│   ├── CreatedResult                   // 201 + data + Location header
│   ├── CreatedAtActionResult           // 201 + data + Location via action name
│   ├── CreatedAtRouteResult            // 201 + data + Location via route name
│   ├── AcceptedResult                  // 202 + data
│   ├── BadRequestObjectResult          // 400 + error details
│   ├── NotFoundObjectResult            // 404 + error details
│   ├── ConflictObjectResult            // 409 + error details
│   └── UnprocessableEntityObjectResult // 422 + error details
│
├── ViewResult                          // renders a full Razor view → HTML
├── PartialViewResult                   // renders a partial view → HTML fragment
│
├── JsonResult                          // serializes to JSON directly
├── ContentResult                       // raw string (plain text, HTML, etc.)
│
├── FileResult                          (abstract, file downloads)
│   ├── FileContentResult               // from byte[]
│   ├── FileStreamResult                // from Stream
│   ├── VirtualFileResult               // from IFileProvider path
│   └── PhysicalFileResult              // from absolute file path on disk
│
├── RedirectResult                      // 301/302 to a URL
├── RedirectToActionResult              // 302 to controller/action
├── RedirectToRouteResult               // 302 to a named route
├── RedirectToPageResult                // 302 to a Razor Page
│
├── ChallengeResult                     // triggers authentication challenge
├── ForbidResult                        // triggers authentication forbid
└── SignInResult / SignOutResult         // cookie authentication sign-in/out
```

### The StatusCodeResult vs ObjectResult pattern

This is the key structural insight in the tree:

| Branch | Purpose | Body? | Example |
|---|---|---|---|
| `StatusCodeResult` subtypes | Signal a status with no body | No | `NotFound()` → 404, empty |
| `ObjectResult` subtypes | Signal a status **with** a serialized body | Yes | `NotFound(error)` → 404 + JSON |

The helper methods on `ControllerBase` follow this convention consistently:

```csharp
return NotFound();        // → new NotFoundResult()         : StatusCodeResult
return NotFound(error);   // → new NotFoundObjectResult()   : ObjectResult

return Ok();              // → new OkResult()               : StatusCodeResult
return Ok(product);       // → new OkObjectResult(product)  : ObjectResult

return BadRequest();             // → new BadRequestResult()
return BadRequest(modelState);   // → new BadRequestObjectResult(modelState)
```

No-argument overload → `StatusCodeResult` (empty body). Argument overload → `ObjectResult` (serialized body). This pattern repeats for every status code that has both forms.

```ad-tip
The `ViewResult` branch is what makes `IActionResult` fundamentally richer than `IResult`. Rendering a Razor view requires the view engine, which requires the `ActionContext` to resolve view names relative to the controller. This is functionality that `IResult` **cannot** replicate because it has no `ActionContext`.
```


## The IResult Hierarchy (Minimal APIs)

[[Minimal APIs]] bypass the MVC pipeline entirely. There are no controllers, no action descriptors, and no `ActionContext`. The result types operate directly on `HttpContext`.

### Structure

The `IResult` hierarchy is deliberately simpler. The concrete types live in `Microsoft.AspNetCore.Http.HttpResults` and are mostly `sealed` internal classes. You don't construct them directly -- you use the `Results` or `TypedResults` factory classes.

```
IResult
├── Ok<T>                    // 200 + serialized value
├── Created<T>               // 201 + Location header + value
├── Accepted<T>              // 202 + value
├── NoContent                // 204
├── BadRequest<T>            // 400 + error details
├── NotFound / NotFound<T>   // 404
├── Conflict<T>              // 409
├── UnprocessableEntity<T>   // 422
├── StatusCodeHttpResult     // arbitrary status code
├── JsonHttpResult<T>        // JSON serialization
├── ContentHttpResult        // raw string
├── FileContentHttpResult    // file from byte[]
├── FileStreamHttpResult     // file from Stream
├── PhysicalFileHttpResult   // file from disk path
├── RedirectHttpResult       // redirect
├── PushStreamHttpResult     // streaming response
├── Results<T1, T2, ...>     // union type for OpenAPI (see below)
└── (others)
```

### Factory methods

```csharp
// Results static class (returns IResult)
Results.Ok(product)           // → IResult that writes 200 + JSON
Results.NotFound()            // → IResult that writes 404
Results.Created($"/api/products/{id}", product)  // → 201 + Location + JSON
Results.File(bytes, "application/pdf")           // → file download
Results.Redirect("/login")                       // → 302

// TypedResults static class (returns concrete types like Ok<T>)
TypedResults.Ok(product)      // → Ok<Product> : IResult
TypedResults.NotFound()       // → NotFound : IResult
```

```ad-info
`Results` returns `IResult`. `TypedResults` returns the **concrete type** (e.g., `Ok<Product>`). Prefer `TypedResults` when you want the endpoint metadata system to infer response types for OpenAPI/Swagger documentation automatically.
```

### The Results\<T1, T2\> union type

Minimal APIs have a special generic union type for declaring multiple possible return types:

```csharp
app.MapGet("/products/{id}", (int id) =>
{
    var product = repository.Find(id);
    return product is not null
        ? TypedResults.Ok(product)         // Ok<Product>
        : TypedResults.NotFound();         // NotFound
})
// Return type: Results<Ok<Product>, NotFound>
```

This is the Minimal API equivalent of `ActionResult<T>` in controllers (see [[03 - ActionResult T]]). It gives the OpenAPI generator enough type information to document all possible responses.


## Why You Cannot Mix Them

Controllers and Minimal APIs are different pipelines with different result interfaces. You cannot cross the streams:

| Scenario | What happens |
|---|---|
| Return `Results.Ok()` from a controller action | **Compile error** -- controller expects `IActionResult`, not `IResult` |
| Return `Ok()` (ControllerBase helper) from a Minimal API lambda | **Compile error** -- `Ok()` is an instance method on `ControllerBase`, not available in a lambda |
| Pass `Results.Ok()` where `IActionResult` is expected | **Compile error** -- `IResult` does not implement `IActionResult` |

Same concept, same HTTP semantics, but different type hierarchies because the execution pipelines are incompatible.

```ad-warning
The naming overlap is intentional and can be confusing. Both hierarchies have types called `Ok`, `NotFound`, `BadRequest`, etc. But `OkObjectResult` (MVC) and `Ok<T>` (Minimal API) are completely unrelated types in different namespaces. Don't assume they're interchangeable.
```


## Side-by-Side Comparison

| Aspect | IActionResult (MVC) | IResult (Minimal APIs) |
|---|---|---|
| **Execution method** | `ExecuteResultAsync(ActionContext)` | `ExecuteAsync(HttpContext)` |
| **Available context** | `ActionContext` (includes `HttpContext`) | `HttpContext` only |
| **Can render Razor views** | Yes (`ViewResult`, `PartialViewResult`) | No |
| **Can access ModelState** | Yes | No |
| **Can access action filters** | Yes | No |
| **Factory surface** | `ControllerBase` instance methods | `Results` / `TypedResults` static classes |
| **OpenAPI union type** | `ActionResult<T>` | `Results<T1, T2, ...>` |
| **Concrete types** | Public, well-known class names | Mostly sealed, accessed via factories |
| **Used by** | [[04 - Controller vs ControllerBase\|Controllers]], Razor Pages | [[Minimal APIs]], raw endpoint routing |
| **Introduced** | ASP.NET Core 1.0 | ASP.NET Core 6.0 (.NET 6) |

```ad-note
The `[ApiController]` attribute (see [[05 - The ApiController Attribute]]) does not change which result hierarchy you use. API controllers still use `IActionResult` -- `[ApiController]` just adds automatic model validation, binding source inference, and `ProblemDetails` responses on top of the existing MVC pipeline.
```


## When Each Hierarchy Applies

Use this as a decision guide based on which programming model you chose (see [[14 - The Four Programming Models]]):

- **MVC controllers** (`Controller` or `ControllerBase`) → `IActionResult` and its subtypes. You get views, model state, filters, and the full MVC pipeline.
- **Minimal APIs** (`app.MapGet`, `app.MapPost`, etc.) → `IResult` via `Results` or `TypedResults`. Lighter weight, no MVC overhead, but no view rendering.
- **Razor Pages** → `IActionResult` (the page model's `OnGet`/`OnPost` handlers return `IActionResult`).

```ad-tip
If you're building a pure API with no views, the choice between controllers and Minimal APIs is architectural, not about result types. Both can return the same HTTP responses (same status codes, same JSON bodies). The difference is whether you want the MVC pipeline's features (filters, model binding conventions, `[ApiController]` behaviors) or prefer the lighter endpoint routing model.
```
