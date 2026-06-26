---
title: "Routing Overview"
date: 2026-06-18
tags: [csharp, asp-net-core, routing, fundamentals]
aliases: [ASP.NET Core Routing, Routing Basics, URL Routing]
status: complete
---

# Routing Overview

> [!ad-note] Overview
> Routing is the mechanism ASP.NET Core uses to match incoming HTTP requests to executable endpoint handlers. It maps URLs to controllers, Razor Pages, minimal API delegates, and other endpoints. This note covers the foundational concepts: route templates, the historical evolution of routing systems, and how routing integrates with the middleware pipeline.

---

## Table of Contents

- [What Is Routing](#What%20Is%20Routing)
- [Historical Context: Two Routing Systems](#Historical%20Context%20Two%20Routing%20Systems)
- [Endpoint Routing: The Unified Model](#Endpoint%20Routing%20The%20Unified%20Model)
- [Route Template Syntax](#Route%20Template%20Syntax)
- [How Routing Integrates with Middleware](#How%20Routing%20Integrates%20with%20Middleware)
- [URL Matching: How ASP.NET Core Chooses an Endpoint](#URL%20Matching%20How%20ASP.NET%20Core%20Chooses%20an%20Endpoint)
- [Comprehensive Summary](#Comprehensive%20Summary)
- [Related Topics](#Related%20Topics)

---

## What Is Routing

**Routing** is the process of matching an incoming HTTP request (its URL path, HTTP method, and sometimes other data) to an **endpoint handler** -- the code that will process the request and produce a response.

In ASP.NET Core, endpoint handlers can be:

- **Controller actions** (MVC pattern)
- **Razor Page handlers** (page-based model)
- **Minimal API delegates** (lambda-based endpoints introduced in .NET 6)
- **gRPC services**, **SignalR hubs**, **health check endpoints**, and more

Routing answers the question: *"Given this incoming URL and HTTP method, which piece of code should handle it?"*

> [!ad-note] Definition
> **Endpoint**: A unit of request-handling logic in ASP.NET Core. Each endpoint has a **route pattern** (the URL template), **metadata** (authorization policies, CORS, etc.), and a **request delegate** (the code that runs).

### Why Routing Matters

Without routing, you would have to write manual URL parsing logic in every application. Routing provides:

- **Declarative URL patterns** -- define what URLs look like, not how to parse them
- **Parameter extraction** -- automatically pull values from URL segments into method parameters
- **URL generation** -- create URLs from route data (the reverse of matching)
- **Centralized configuration** -- all URL-to-handler mappings are visible in one place
- **Middleware integration** -- authorization, CORS, and rate limiting can inspect the selected endpoint before it executes

> [!summary] Section Summary
> - Routing matches HTTP requests to endpoint handlers based on URL patterns.
> - Endpoints include controller actions, Razor Pages, minimal APIs, gRPC, SignalR, and more.
> - Routing provides parameter extraction, URL generation, and middleware integration.
> - Each endpoint has a route pattern, metadata, and a request delegate.

---

## Historical Context: Two Routing Systems

Understanding ASP.NET Core routing requires knowing where it came from. Historically, two distinct systems coexisted.

### Conventional Routing (MVC Legacy)

Conventional routing (also called **convention-based routing**) defines routes in a centralized location, typically in `Program.cs` (or the old `Startup.cs`). Routes are defined as templates that map URL segments to controller/action names by convention:

```csharp
// Conventional route definition
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

This single route handles URLs like:
| URL | Controller | Action | id |
|---|---|---|---|
| `/` | `Home` | `Index` | `null` |
| `/Products` | `Products` | `Index` | `null` |
| `/Products/Details/5` | `Products` | `Details` | `5` |

**Characteristics:**
- Routes are defined *away* from the controllers
- Relies on naming conventions (`{controller}` and `{action}` tokens)
- Good for MVC applications with predictable URL structures
- Hard to see which URLs a controller responds to without checking the route table

### Attribute Routing

**[[Attribute Routing]]** defines routes directly on controllers and actions using attributes like `[Route]`, `[HttpGet]`, and `[HttpPost]`:

```csharp
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetProduct(int id) => Ok();
}
```

**Characteristics:**
- Routes live *next to* the code they describe
- Explicit -- no convention magic
- Required for Web API controllers (the `[ApiController]` attribute mandates it)
- Each action clearly declares its own URL pattern

### The Tension

Before .NET Core 3.0, these two systems had separate internal implementations. Routing happened *inside* MVC, which meant middleware running before MVC had no way to know which endpoint would be selected. This created real problems for cross-cutting concerns like authorization and CORS, which needed to know the target endpoint to make decisions.

> [!warning] Common Misconception
> Conventional routing and attribute routing are **not** mutually exclusive. You can use both in the same application. Controllers without `[Route]` attributes use conventional routes; controllers with them use attribute routes. However, a single action cannot participate in both systems simultaneously.

> [!summary] Section Summary
> - Conventional routing defines routes centrally using `{controller}/{action}` templates.
> - Attribute routing defines routes on controllers and actions via attributes.
> - Before .NET Core 3.0, both systems had separate internal implementations.
> - Middleware before MVC could not inspect which endpoint was selected -- a major limitation.
> - Both systems can coexist in the same application.

---

## Endpoint Routing: The Unified Model

**.NET Core 3.0** introduced **[[Endpoint Routing]]** -- a unified routing system that decoupled route matching from endpoint execution. This is the system all modern ASP.NET Core applications use.

### The Two-Phase Model

Endpoint routing splits routing into two distinct phases:

1. **Route matching** (`UseRouting()`) -- examines the incoming URL and selects the best matching endpoint. The selected endpoint is attached to the `HttpContext` but *not yet executed*.

2. **Endpoint execution** (the terminal middleware like `MapControllers()`) -- runs the selected endpoint's request delegate.

```csharp
var app = builder.Build();

app.UseRouting();           // Phase 1: Select endpoint
app.UseAuthentication();    // Runs AFTER endpoint is selected
app.UseAuthorization();     // Can inspect the selected endpoint's metadata
// Phase 2: Execute the selected endpoint (implicitly at the end of the pipeline)
app.MapControllers();
app.MapRazorPages();
```

> [!ad-note] Key Insight
> The power of this two-phase model is that **middleware between `UseRouting()` and the endpoint execution can inspect the selected endpoint's metadata**. Authorization middleware, for example, can check whether the endpoint requires authentication *before* the endpoint code runs.

### Why This Matters

Before endpoint routing:
- Authorization had to run *inside* MVC filters, not in the middleware pipeline
- CORS decisions could not be made until MVC processed the request
- There was no way for generic middleware to know what code would eventually handle the request

After endpoint routing:
- Any middleware can call `context.GetEndpoint()` to inspect the selected endpoint
- Authorization, CORS, rate limiting, and other middleware work uniformly across all endpoint types
- The same routing system serves controllers, Razor Pages, minimal APIs, gRPC, and SignalR

> [!summary] Section Summary
> - Endpoint routing (since .NET Core 3.0) unifies all routing into a two-phase model.
> - Phase 1 (`UseRouting()`) selects the endpoint; Phase 2 executes it.
> - Middleware between the two phases can inspect endpoint metadata (authorization, CORS, etc.).
> - This design enables cross-cutting concerns to work uniformly across all endpoint types.

---

## Route Template Syntax

Route templates define the URL patterns that routing matches against. They are strings composed of **literal segments**, **route parameters**, and special syntax.

### Literal Segments

Fixed text that must match exactly:

```
"products/catalog"
```

Matches: `/products/catalog`
Does not match: `/products/list` or `/items/catalog`

### Route Parameters

Placeholders enclosed in curly braces that capture values from the URL:

```
"products/{id}"
```

Matches: `/products/5`, `/products/abc`, `/products/anything-here`
The captured value is available as a route value named `id`.

### Optional Parameters

Append `?` to make a parameter optional:

```
"products/{id?}"
```

Matches: `/products` (id is null) and `/products/5` (id is "5")

### Default Values

Use `=` to specify a default when the segment is absent:

```
"{controller=Home}/{action=Index}/{id?}"
```

| URL | controller | action | id |
|---|---|---|---|
| `/` | `Home` | `Index` | `null` |
| `/Products` | `Products` | `Index` | `null` |
| `/Products/Edit/3` | `Products` | `Edit` | `3` |

### Catch-All Parameters

Prefix with `*` or `**` to capture the rest of the URL (including slashes):

```
"files/{**path}"
```

| URL | path |
|---|---|
| `/files/docs/readme.txt` | `docs/readme.txt` |
| `/files/images/2026/photo.jpg` | `images/2026/photo.jpg` |
| `/files/` | `` (empty string) |

> [!ad-note] Important
> `{*path}` and `{**path}` behave identically. The double-asterisk form is preferred for clarity and is the documented convention.

### Route Constraints

Constraints restrict which values a parameter will match. They are appended with a colon:

```
"products/{id:int}"
```

Matches: `/products/5` (id is an integer)
Does not match: `/products/abc` (not an integer, routing moves to the next route)

See [[Route Constraints and Tokens]] for the full list of built-in constraints.

### Complex Segments

Parameters can be mixed with literals in a single segment:

```
"files/{filename}.{ext}"
```

Matches: `/files/report.pdf` (filename = "report", ext = "pdf")

### Full Syntax Reference Table

| Syntax | Meaning | Example |
|---|---|---|
| `literal` | Exact match | `products` |
| `{param}` | Required parameter | `{id}` |
| `{param?}` | Optional parameter | `{id?}` |
| `{param=default}` | Default value | `{action=Index}` |
| `{**param}` | Catch-all (greedy) | `{**slug}` |
| `{param:constraint}` | Constrained parameter | `{id:int}` |
| `{param:c1:c2}` | Multiple constraints | `{id:int:min(1)}` |

> [!warning] Common Misconception
> Optional parameters and default values are **not** the same thing. `{id?}` means the segment can be missing and `id` will be `null`. `{id=5}` means if the segment is missing, `id` will be `"5"`. You can combine them: `{id:int?}` means optional with a constraint, but no default.

> [!summary] Section Summary
> - Route templates combine literal segments and parameters in curly braces.
> - Parameters can be required, optional (`?`), have defaults (`=value`), or be catch-all (`**`).
> - Constraints restrict which values match a parameter (e.g., `{id:int}`).
> - Complex segments can mix literals and parameters within a single URL segment.
> - Catch-all parameters capture everything including forward slashes.

---

## How Routing Integrates with Middleware

Routing is not a standalone system -- it is deeply integrated into the ASP.NET Core **middleware pipeline**. Understanding this integration is essential for building correct applications.

### The Middleware Pipeline

ASP.NET Core processes each request through an ordered sequence of middleware components. Each middleware can:
- Handle the request and short-circuit the pipeline
- Pass the request to the next middleware
- Run logic before and/or after the next middleware

### Routing's Place in the Pipeline

```csharp
var app = builder.Build();

// Middleware that runs BEFORE routing (no endpoint info available)
app.UseHttpsRedirection();
app.UseStaticFiles();      // Can short-circuit for static files

// ROUTING: selects the endpoint
app.UseRouting();

// Middleware between routing and endpoint execution
// These CAN inspect the selected endpoint
app.UseCors();
app.UseAuthentication();
app.UseAuthorization();
app.UseRateLimiter();

// ENDPOINT EXECUTION: runs the selected endpoint
app.MapControllers();
app.MapRazorPages();
app.MapGet("/hello", () => "Hello World");
```

### The Three Zones

Think of the pipeline as three zones:

| Zone | Position | Endpoint Available? | Use For |
|---|---|---|---|
| Pre-routing | Before `UseRouting()` | No | Static files, HTTPS redirect, exception handling |
| Between routing and execution | After `UseRouting()`, before `Map*` | Yes (selected but not executed) | Auth, CORS, rate limiting -- anything that needs to inspect the endpoint |
| Post-routing | Inside `Map*` handlers | Yes (executing) | The actual request handling |

### Implicit UseRouting

> [!tip] Practical Tip
> In .NET 6+ with minimal hosting, `UseRouting()` is called **implicitly** if you do not call it yourself. The framework inserts it at the correct position. However, if you need middleware to run between routing and endpoint execution (which is common), you should call `UseRouting()` explicitly to control the position.

### Accessing the Selected Endpoint in Middleware

Any middleware after `UseRouting()` can inspect the selected endpoint:

```csharp
app.Use(async (context, next) =>
{
    var endpoint = context.GetEndpoint();

    if (endpoint != null)
    {
        // The endpoint was selected -- inspect its metadata
        var routeName = endpoint.DisplayName;
        var requiresAuth = endpoint.Metadata.GetMetadata<AuthorizeAttribute>();
    }
    else
    {
        // No endpoint matched this request
    }

    await next(context);
});
```

> [!warning] Common Misconception
> `UseRouting()` does **not** execute the endpoint. It only selects which endpoint matches. The endpoint runs later, at the end of the pipeline. If no endpoint matches, `context.GetEndpoint()` returns `null` and the request falls through to a 404.

> [!summary] Section Summary
> - The middleware pipeline has three zones: pre-routing, between routing and execution, and post-routing.
> - `UseRouting()` selects the endpoint; `Map*` methods execute it at the end of the pipeline.
> - Middleware between the two phases can inspect the selected endpoint and its metadata.
> - In .NET 6+, `UseRouting()` is implicit but should be called explicitly when you need to control middleware ordering.
> - `context.GetEndpoint()` returns the selected endpoint or `null` if no route matched.

---

## URL Matching: How ASP.NET Core Chooses an Endpoint

When multiple route templates could potentially match an incoming URL, ASP.NET Core uses a set of rules to determine the **best match**.

### Matching Priority

The routing system ranks candidates using **specificity**:

1. **Literal segments** are more specific than parameters
2. **Constrained parameters** are more specific than unconstrained ones
3. **Required parameters** are more specific than optional ones
4. **Catch-all parameters** are the least specific

For example, given these routes:
```
"products/featured"          // All literals
"products/{category}"        // One parameter
"products/{category}/{id:int}" // Two params, one constrained
"{**catchall}"               // Catch-all
```

The URL `/products/featured` matches the first route (literal match wins).
The URL `/products/electronics` matches the second route.
The URL `/products/electronics/42` matches the third route.
The URL `/anything/else/here` matches the catch-all.

### Ambiguous Routes

If two routes have equal specificity for a given URL, ASP.NET Core throws an **`AmbiguousMatchException`** at runtime:

```csharp
app.MapGet("products/{id}", handler1);
app.MapGet("products/{name}", handler2);
// Both match /products/something -- AMBIGUOUS!
```

> [!danger] Critical Warning
> Ambiguous route exceptions happen at **request time**, not at startup. Your application will start successfully, and the error only surfaces when a request triggers the ambiguity. Always test your route configurations with realistic URLs.

### Route Order

You can influence matching order with:
- **The `Order` property** on attribute routes: `[HttpGet("{id}", Order = 1)]`
- **The order of `MapControllerRoute()` calls** -- earlier registrations take priority
- **Convention: more specific routes first**

> [!tip] Practical Tip
> In most applications, you do not need to set `Order` explicitly. The built-in specificity rules handle the vast majority of cases. If you find yourself frequently setting `Order`, it may be a sign that your route structure needs redesigning.

### HTTP Method Matching

For [[Attribute Routing]], the HTTP method is part of the match. `[HttpGet("{id}")]` only matches GET requests. `[HttpPost]` only matches POST. If the URL matches but the method does not, the framework returns **405 Method Not Allowed** (not 404).

For conventional routing, all HTTP methods match by default unless you add method constraints.

> [!summary] Section Summary
> - ASP.NET Core resolves route ambiguity using specificity: literals > constrained params > required params > optional > catch-all.
> - Equally specific routes cause `AmbiguousMatchException` at request time, not startup.
> - Route order can be influenced with `Order` or registration order, but specificity usually suffices.
> - HTTP method matching is part of attribute routing; mismatches produce 405 (not 404).

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Routing** in ASP.NET Core is the system that matches incoming HTTP requests to endpoint handlers. It evolved from two separate systems -- **conventional routing** (centralized templates with `{controller}/{action}` conventions) and **[[Attribute Routing]]** (route attributes on controllers/actions) -- into a unified **[[Endpoint Routing]]** system introduced in .NET Core 3.0.
>
> Endpoint routing uses a **two-phase model**: `UseRouting()` selects the best matching endpoint, and the terminal middleware (`MapControllers()`, `MapRazorPages()`, `MapGet()`, etc.) executes it. Middleware between these two phases can inspect the selected endpoint's metadata, enabling authorization, CORS, and rate limiting to work uniformly across all endpoint types.
>
> **Route templates** define URL patterns using literal segments, parameters (`{id}`), optional parameters (`{id?}`), defaults (`{action=Index}`), catch-all parameters (`{**slug}`), and **[[Route Constraints and Tokens|constraints]]** (`{id:int}`). When multiple routes match, ASP.NET Core resolves the conflict using specificity rules where literals beat parameters and constrained parameters beat unconstrained ones.
>
> The middleware pipeline has three zones relative to routing: pre-routing (no endpoint info), between routing and execution (endpoint selected but not run), and execution (the endpoint runs). In .NET 6+, `UseRouting()` is implicit but should be called explicitly when precise middleware ordering matters.

---

## Related Topics

- [[Attribute Routing]]
- [[Endpoint Routing]]
- [[Route Constraints and Tokens]]
- [[Middleware Pipeline]]
- [[Minimal APIs]]
- [[Model Binding]]
- [[URL Generation]]
- [[Areas in MVC]]

---

## Further Reading

- [[Controllers and Actions]]
- [[Razor Pages]]
- [[Web API Design]]
- [[Authentication and Authorization]]
- [[CORS in ASP.NET Core]]
