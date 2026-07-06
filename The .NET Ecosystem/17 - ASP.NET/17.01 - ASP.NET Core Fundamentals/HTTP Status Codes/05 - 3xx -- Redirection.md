---
tags: [csharp, asp-net-core, http, status-codes, web-api, fundamentals]
---


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
