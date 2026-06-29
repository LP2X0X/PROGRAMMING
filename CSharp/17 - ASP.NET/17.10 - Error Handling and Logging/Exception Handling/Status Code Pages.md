---
tags:
  - csharp
  - asp-net-core
  - error-handling
  - exceptions
  - middleware
---


Not all errors involve exceptions. A 404 (Not Found), 403 (Forbidden), or other non-success status code might be set by a controller or routing middleware without throwing an exception. The **Status Code Pages** middleware handles these non-exception error status codes.

## UseStatusCodePages

The simplest form provides a plain-text response for any non-success, non-redirect status code:

```csharp
app.UseStatusCodePages();
// Returns: "Status Code: 404; Not Found"
```

You can customize the response with a format string:

```csharp
app.UseStatusCodePages("text/plain", "Error: status code {0}");
```

## UseStatusCodePagesWithReExecute

This is the most useful variant. It re-executes the pipeline to a specified path, allowing you to render a full error page with your layout:

```csharp
// The {0} placeholder is replaced with the status code
app.UseStatusCodePagesWithReExecute("/errors/{0}");
```

```csharp
public class ErrorsController : Controller
{
    [Route("/errors/{statusCode}")]
    public IActionResult HandleStatusCode(int statusCode)
    {
        return statusCode switch
        {
            404 => View("NotFound"),   // ~/Views/Errors/NotFound.cshtml
            403 => View("Forbidden"),  // ~/Views/Errors/Forbidden.cshtml
            _ => View("GenericError", new ErrorViewModel { StatusCode = statusCode })
        };
    }
}
```

## UseStatusCodePagesWithRedirects

This variant issues a **302 redirect** to the error page. Generally **avoid** this because:

- The browser URL changes to `/errors/404`, which is misleading
- The original URL is lost
- The status code sent to the browser is 302, then 200 (not the original 404) -- bad for SEO and client-side error handling

```csharp
// Generally avoid this -- prefer ReExecute
app.UseStatusCodePagesWithRedirects("/errors/{0}");
```

## Important Placement

Status code pages middleware must be placed **early** in the pipeline, but **after** the exception handler:

```csharp
app.UseExceptionHandler("/Home/Error");  // Catches exceptions
app.UseStatusCodePages();                 // Catches non-exception error status codes
app.UseStaticFiles();
app.UseRouting();
```

> [!warning] Common Misconception
> Status code pages middleware does **not** handle exceptions. It handles status codes set by downstream middleware and endpoints when no response body has been written. If a controller returns `NotFound()` (which sets 404 but writes no body), the status code pages middleware generates the error response. But if a controller throws a `KeyNotFoundException`, that is caught by the exception handler, not the status code pages middleware.

> [!ad-note]
> `UseStatusCodePagesWithReExecute` preserves the original URL in the browser address bar and returns the correct status code to the client (e.g., 404). This is the preferred approach for SEO and proper client-side error handling. The re-execute is internal, just like `UseExceptionHandler`.

> [!summary] Section Summary
> - Status code pages middleware handles non-exception error responses (404, 403, etc.) where no response body was written
> - `UseStatusCodePagesWithReExecute("/errors/{0}")` is the preferred variant -- it preserves the URL and returns the correct status code
> - Avoid `UseStatusCodePagesWithRedirects` because it changes the URL and loses the original status code
> - Place status code pages middleware after exception handling but before routing
> - This middleware complements, not replaces, exception handling middleware
