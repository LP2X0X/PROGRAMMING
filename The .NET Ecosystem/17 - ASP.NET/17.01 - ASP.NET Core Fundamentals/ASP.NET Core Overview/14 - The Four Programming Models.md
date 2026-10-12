---
tags: [csharp, asp-net-core, fundamentals, web]
aliases: [ASP.NET Core Programming Models, MVC vs Razor Pages vs Minimal APIs]
---


ASP.NET Core ships four distinct programming models for handling HTTP requests. They are not abstraction layers stacked on top of each other -- they are **parallel approaches** that share some infrastructure but solve different problems.

---

## Quick Comparison

| | MVC Controller | Razor Pages | API Controller | Minimal API |
|---|---|---|---|---|
| **Returns** | HTML (via Razor view) | HTML (via Razor view) | Data (JSON/XML) | Data (JSON/XML) |
| **Base class** | `Controller` | `PageModel` | `ControllerBase` | None (lambda/delegate) |
| **Routing** | Conventional or attribute | Convention (`/PageName`) | Attribute routing | Inline `app.MapXxx()` |
| **Unit of work** | Action method on a controller class | Handler method (`OnGet`, `OnPost`) on a page model | Action method on a controller class | A delegate/lambda |
| **View engine** | Razor (`.cshtml`) | Razor (`.cshtml`) colocated | None | None |
| **Distinguishing trait** | Full MVC pattern with separate Views folder | Page + model colocated in one place | [[05 - The ApiController Attribute\|`[ApiController]`]] behaviors | Zero ceremony, no class needed |

---

## Architecture Relationship

A common misconception is that these four models form layers, with Minimal APIs at the bottom and MVC at the top. That is wrong. They are **siblings** attached to the same routing foundation.

```
Endpoint Routing (the shared foundation)
│
├── Minimal APIs          (lambda → response, no MVC pipeline)
│
└── MVC Middleware         (UseRouting + MapControllers / MapRazorPages)
    │
    ├── API Controller     (class → data response, no view engine)
    │
    ├── MVC Controller     (class → Razor view → HTML response)
    │
    └── Razor Pages        (page model → Razor view → HTML response)
```

### What the MVC Middleware branch shares

The three models under MVC Middleware go through the **same pipeline**:

- [[17.06 - Controllers and Actions\|Action/page filters]] (authorization, resource, action, result, exception)
- **Model binding** -- request data mapped to method parameters or page properties
- **Model validation** -- `[Required]`, `[Range]`, `ModelState.IsValid`
- **Content negotiation** (API Controllers only)
- **Dependency injection** via constructor

Razor Pages is built on the MVC infrastructure internally. It uses the same `IActionResult` system, the same filter pipeline, and the same Razor view engine. Microsoft's source code has Razor Pages implemented as a special case of MVC routing.

### Where Minimal APIs diverge

Minimal APIs bypass the MVC middleware entirely. They have their own pipeline:

- **Endpoint filters** (not MVC filters -- a separate, simpler concept)
- **Parameter binding** from route, query, body, services -- but no `ModelState`
- **No base class** -- endpoints are delegates, not methods on a class
- DI via method parameters, not constructor injection

> [!warning] Common Misconception
> Minimal APIs are not "lightweight MVC controllers." They share zero runtime code with MVC. They connect to the same endpoint routing system, but everything after route matching is different.

---

## Why Each Exists -- Historical Context

Each programming model was created to solve a specific friction that the previous generation introduced.

### MVC Controller (2009 -- ASP.NET MVC 1.0)

**Problem**: ASP.NET Web Forms pretended HTTP was stateful. ViewState, postbacks, and the page lifecycle made server-rendered apps feel like desktop apps, but the abstraction leaked everywhere. The industry (Rails, Django, Spring MVC) had already moved to proper separation of concerns.

**Solution**: Adopt the Model-View-Controller pattern. The controller handles the request, builds a model, and selects a view. Clean separation, testable, no ViewState.

### Web API Controller (2012 -- ASP.NET Web API, later merged into Core)

**Problem**: Mobile apps and JavaScript SPAs arrived. They do not need HTML pages -- they need raw data (JSON). MVC controllers *could* return JSON, but they carried view engine baggage. Content negotiation, proper HTTP status codes, and `Accept` header handling were afterthoughts.

**Solution**: A controller base class (`ControllerBase` instead of `Controller`) stripped of view support. The [[05 - The ApiController Attribute\|`[ApiController]`]] attribute adds automatic model validation, binding source inference, and `ProblemDetails` error responses.

### Razor Pages (2017 -- ASP.NET Core 2.0)

**Problem**: MVC forces you to split every page across two locations -- a controller in `/Controllers/` and a view in `/Views/`. For page-centric applications (settings pages, forms, dashboards), this is busywork. You end up with controllers that have a single action method.

**Solution**: Colocate the handler and the view. `Settings.cshtml` and `Settings.cshtml.cs` live side by side. The `PageModel` class contains `OnGet()` and `OnPost()` handlers. No separate controller needed.

### Minimal APIs (2021 -- .NET 6)

**Problem**: Express.js, Flask, and Go let you define an API endpoint in a single expression. ASP.NET Core required a class with inheritance, constructor injection, attributes, and a startup file. For microservices returning a few JSON endpoints, that was too much ceremony.

**Solution**: Define endpoints as lambdas registered directly on the `WebApplication` object. No class, no inheritance, no attributes.

### The Pattern

Each model was a reaction to the previous one adding more structure than a use case needed.

| Shift | Response |
|---|---|
| Web Forms is too coupled and stateful | MVC Controller |
| Mobile/SPA needs data, not HTML pages | API Controller |
| One-action controllers are busywork | Razor Pages |
| A class per endpoint is too heavy | Minimal APIs |

---

## One-Line Code Comparison

```csharp
// MVC Controller -- returns an HTML view with a model
public IActionResult Details(int id) => View(product);

// Razor Pages -- handler method on the page model
public IActionResult OnGet(int id) { Product = _repo.Get(id); return Page(); }

// API Controller -- returns data with HTTP 200
[HttpGet("{id}")]
public IActionResult GetProduct(int id) => Ok(product);

// Minimal API -- inline endpoint definition, no class
app.MapGet("/api/products/{id}", (int id, IProductRepo repo) => Results.Ok(repo.Get(id)));
```

Key differences visible here:

- MVC Controller calls `View()` -- it needs a matching `.cshtml` file
- Razor Pages calls `Page()` -- the `.cshtml` file is colocated and implied
- API Controller calls `Ok()` -- serializes to JSON, returns 200
- Minimal API uses `Results.Ok()` -- static helper, same JSON result but no class

---

## Choosing Between Them

### Step 1: HTML or Data?

This is the first and most important decision.

| Need | Options |
|---|---|
| **Server-rendered HTML** | MVC Controller or Razor Pages |
| **Data API (JSON/XML)** | API Controller or Minimal API |

### Step 2: How Much Structure?

Within each pair, the choice is about how much framework scaffolding you want.

**For HTML applications:**

| Factor | MVC Controller | Razor Pages |
|---|---|---|
| Best for | Complex apps with shared controllers across many views | Page-centric apps (forms, dashboards, CRUD) |
| File organization | Controllers in one folder, views in another | Page + model colocated |
| Routing | [[09 - Attribute Routing vs Conventional Routing\|Conventional or attribute routing]] | Convention-based (`/Pages/Products/Index.cshtml` maps to `/Products`) |
| When to pick | Multiple actions per controller, heavy use of `ViewData`/`TempData`, complex routing | Most new server-rendered apps, simpler mental model |

**For data APIs:**

| Factor | API Controller | Minimal API |
|---|---|---|
| Best for | Large APIs with many endpoints, teams, OpenAPI generation | Microservices, small APIs, prototypes |
| Model validation | Automatic via `[ApiController]` + `ModelState` | Manual, or via endpoint filters (.NET 7+) |
| Filters | Full MVC filter pipeline (authorization, action, exception, result) | Endpoint filters (simpler, fewer hook points) |
| Content negotiation | Built-in (`Accept` header, formatters) | Not built-in (returns JSON by default) |
| Grouping | Controllers are natural groups | `MapGroup()` (added in .NET 7) |
| API versioning | Mature support | Supported from .NET 7 |
| Testability | Instantiate controller class, inject mocks | Call the delegate directly or use `WebApplicationFactory` |
| OpenAPI / Swagger | Full `[ProducesResponseType]` support | Supported via `Produces<T>()` and `WithOpenApi()` |

> [!tip] They Coexist
> All four models can run in the same application. You can have Razor Pages for your admin dashboard, API controllers for your main API surface, and a few Minimal API endpoints for health checks -- all in one project. Endpoint routing dispatches to the correct handler regardless of the model.

### Decision Flowchart

```
Do you need to return HTML?
├── Yes → Is the app page-centric (forms, settings, CRUD)?
│         ├── Yes → Razor Pages
│         └── No (complex routing, many views per controller) → MVC Controller
│
└── No (returning data) → How large is the API surface?
          ├── Small / microservice / prototype → Minimal API
          └── Large / team-maintained / needs full filter pipeline → API Controller
```

---

## Mixing Models -- Practical Registration

All four models register through `WebApplication`, but they use different service and endpoint registrations:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Services
builder.Services.AddControllersWithViews();  // MVC Controllers + Views
builder.Services.AddRazorPages();            // Razor Pages
// API Controllers are included with AddControllers() or AddControllersWithViews()
// Minimal APIs need no service registration

var app = builder.Build();

app.MapControllers();          // MVC + API Controllers (attribute-routed)
app.MapDefaultControllerRoute(); // MVC conventional route: {controller}/{action}/{id?}
app.MapRazorPages();           // Razor Pages

// Minimal APIs -- registered inline
app.MapGet("/healthz", () => Results.Ok("healthy"));

app.Run();
```

> [!warning] AddControllers vs AddControllersWithViews
> `AddControllers()` registers controller services **without** the Razor view engine -- use it for API-only projects. `AddControllersWithViews()` adds Razor support on top. If you only call `AddControllers()`, calling `View()` from a controller will fail at runtime.

---

## Summary

| Aspect | MVC Controller | Razor Pages | API Controller | Minimal API |
|---|---|---|---|---|
| **Pipeline** | MVC Middleware | MVC Middleware | MVC Middleware | Endpoint routing (direct) |
| **Returns** | HTML | HTML | JSON/XML | JSON/XML |
| **Ceremony** | High | Medium | High | Low |
| **Filter system** | MVC filters | MVC filters | MVC filters | Endpoint filters |
| **Model binding** | Full | Full | Full | Simplified |
| **Ideal for** | Complex server-rendered apps | Page-centric apps | Large/team APIs | Microservices, small APIs |
| **Introduced** | 2009 | 2017 | 2012 | 2021 |

> [!summary] Key Takeaways
> - The four models are **parallel**, not layered. They all connect to endpoint routing.
> - MVC Controller, API Controller, and Razor Pages share the MVC middleware pipeline (filters, model binding, validation). Minimal APIs have their own simpler pipeline.
> - Razor Pages is built on MVC internally -- it is MVC with a page-centric file organization.
> - API Controllers and Minimal APIs both return data, but API Controllers provide more built-in structure (filters, content negotiation, automatic validation).
> - Choose HTML vs data first, then choose how much structure you want.
> - All four models can coexist in a single application.

---

## Related Topics

- [[17.06 - Controllers and Actions]]
- [[Controllers Overview]]
- [[17.07 - Views and Razor]]
- [[17.08 - Web APIs]]
- [[API Controllers]]
- [[Minimal APIs]]
- [[04 - Controller vs ControllerBase]]
- [[09 - Attribute Routing vs Conventional Routing]]
- [[05 - The ApiController Attribute]]
- [[10 - The Request Pipeline at a Glance]]
- [[08 - What You Can Build]]
