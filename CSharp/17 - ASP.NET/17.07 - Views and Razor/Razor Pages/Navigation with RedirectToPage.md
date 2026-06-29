---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


`RedirectToPage()` is the Razor Pages equivalent of `RedirectToAction()` in MVC:

```csharp
// Redirect to a page in the same folder
return RedirectToPage("./Index");

// Redirect to a page with a route parameter
return RedirectToPage("./Details", new { id = product.Id });

// Redirect to a page in a different folder
return RedirectToPage("/Products/Index");

// Redirect to a named handler
return RedirectToPage("./Edit", "Save", new { id = product.Id });
// URL: /Products/Edit/42?handler=Save

// Redirect to the current page (refresh)
return RedirectToPage();

// Redirect with a fragment
return RedirectToPage("./Details", null, new { id = product.Id }, "reviews");
// URL: /Products/Details/42#reviews
```

### Generating URLs (Without Redirecting)

In the Razor view, use [[Tag Helpers]]:

```cshtml
<a asp-page="./Details" asp-route-id="@product.Id">View Details</a>
<a asp-page="/Products/Index">All Products</a>
<a asp-page="./Edit" asp-route-id="@product.Id" asp-page-handler="Delete">Delete</a>
```

> [!ad-note] Relative vs Absolute Page Paths
> `"./Index"` is relative to the current page's folder. `"/Products/Index"` is an absolute path from the Pages root. Relative paths are recommended within the same feature folder for maintainability.

> [!summary] Section Summary
> - `RedirectToPage("./PageName")` navigates to another Razor Page after processing
> - Use relative paths (`./`) within the same folder, absolute paths (`/Folder/Page`) across folders
> - Route parameters are passed via anonymous objects: `new { id = 42 }`
> - In views, `asp-page` and `asp-route-{param}` generate links to Razor Pages

---
