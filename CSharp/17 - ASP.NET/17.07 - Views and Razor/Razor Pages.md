---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---

# Razor Pages

> [!ad-note] Overview
> Razor Pages is a page-focused programming model in ASP.NET Core that simplifies building server-rendered web UIs. Instead of the MVC pattern of separate controllers, actions, and views, Razor Pages combines the page (`.cshtml`) and its code-behind (`PageModel` in `.cshtml.cs`) into a single, self-contained unit. It is ideal for form-heavy applications, CRUD pages, and scenarios where MVC's indirection adds more ceremony than value.

## Table of Contents

- [What Razor Pages Are](#what-razor-pages-are)
- [Razor Pages vs MVC Controllers](#razor-pages-vs-mvc-controllers)
- [File Structure and URL Routing](#file-structure-and-url-routing)
- [The @page Directive](#the-page-directive)
- [The PageModel Class](#the-pagemodel-class)
- [Handler Methods](#handler-methods)
- [Named Handlers](#named-handlers)
- [Model Binding with BindProperty](#model-binding-with-bindproperty)
- [Page Routing and Route Parameters](#page-routing-and-route-parameters)
- [Navigation with RedirectToPage](#navigation-with-redirecttopage)
- [Layouts, Partials, and View Components](#layouts-partials-and-view-components)
- [Advantages of Razor Pages](#advantages-of-razor-pages)
- [Limitations of Razor Pages](#limitations-of-razor-pages)
- [Real-World Example: Product CRUD Pages](#real-world-example-product-crud-pages)
- [Comprehensive Summary](#comprehensive-summary)
- [Related Topics](#related-topics)

---

## What Razor Pages Are

**Razor Pages** is a server-side, page-centric framework built on top of ASP.NET Core MVC. Each "page" is a pair of files:

1. `Index.cshtml` -- the Razor view with HTML and [[Razor Syntax]]
2. `Index.cshtml.cs` -- the **PageModel** class (the code-behind) containing handler methods and properties

The two files together form a self-contained unit that handles requests and renders responses for a specific URL.

```
/Pages/
    /Products/
        Index.cshtml        ← the page (Razor view)
        Index.cshtml.cs     ← the PageModel (C# code-behind)
        Create.cshtml
        Create.cshtml.cs
        Edit.cshtml
        Edit.cshtml.cs
        Delete.cshtml
        Delete.cshtml.cs
    Index.cshtml
    Index.cshtml.cs
    Privacy.cshtml
    Privacy.cshtml.cs
    _ViewImports.cshtml
    _ViewStart.cshtml
```

Razor Pages was introduced in ASP.NET Core 2.0. It is not a replacement for MVC -- both coexist in the same application. In fact, Razor Pages runs on top of the MVC infrastructure (routing, model binding, filters, etc.).

> [!ad-note] Not Razor Components
> Do not confuse **Razor Pages** (server-rendered, page-focused MVC) with **Razor Components** (Blazor's interactive component model). They share [[Razor Syntax]] but serve very different purposes.

> [!summary] Section Summary
> - Razor Pages pairs a `.cshtml` view with a `.cshtml.cs` PageModel code-behind
> - It is built on top of ASP.NET Core MVC infrastructure
> - Pages are self-contained units handling specific URLs
> - Razor Pages coexists with MVC controllers in the same application

---

## Razor Pages vs MVC Controllers

Choosing between Razor Pages and MVC controllers depends on the nature of the UI:

| Aspect | Razor Pages | MVC Controllers |
|---|---|---|
| **Best for** | Form-heavy pages, CRUD, content sites | Complex routing, APIs, multi-view controllers |
| **Organization** | Page-centric (page + model together) | Feature-centric (controller groups related actions) |
| **File count per feature** | 2 files (`.cshtml` + `.cshtml.cs`) | 3+ files (controller + view + view model) |
| **URL mapping** | File path = URL (convention-based) | Route attributes or conventions |
| **Ceremony** | Low -- minimal boilerplate | Higher -- controller, action, view, routing |
| **Testability** | Good (PageModel is testable) | Good (controller is testable) |
| **Typical app** | Admin panels, settings pages, forms | E-commerce with complex workflows, SPAs, APIs |

> [!tip] Practical Tip
> A common pattern in large applications is to use **Razor Pages for admin/CRUD sections** and **MVC controllers for the public-facing site and APIs**. They coexist naturally. Enable both in `Program.cs`:
> ```csharp
> builder.Services.AddRazorPages();
> builder.Services.AddControllersWithViews();
>
> app.MapRazorPages();
> app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
> ```

> [!summary] Section Summary
> - Razor Pages is best for form-heavy, CRUD, and page-oriented scenarios
> - MVC controllers are better for complex routing, APIs, and multi-view workflows
> - Both can coexist in the same application -- choose per feature, not per application
> - Razor Pages has less ceremony: fewer files and no need for separate routing configuration

---

## File Structure and URL Routing

Razor Pages uses **convention-based routing** where the file path under `/Pages/` directly maps to the URL:

| File Path | URL |
|---|---|
| `/Pages/Index.cshtml` | `/` or `/Index` |
| `/Pages/Privacy.cshtml` | `/Privacy` |
| `/Pages/Products/Index.cshtml` | `/Products` or `/Products/Index` |
| `/Pages/Products/Create.cshtml` | `/Products/Create` |
| `/Pages/Products/Edit.cshtml` | `/Products/Edit` |
| `/Pages/Account/Login.cshtml` | `/Account/Login` |

**Key conventions:**
- The `/Pages/` folder is the root (not included in the URL)
- `Index.cshtml` is the default document for a folder
- Folder hierarchy = URL hierarchy
- The `.cshtml` extension is not included in the URL

### Changing the Pages Root Folder

```csharp
builder.Services.AddRazorPages().AddRazorPagesOptions(options =>
{
    options.RootDirectory = "/Content";  // Instead of /Pages
});
```

### Adding Area-Based Razor Pages

```csharp
builder.Services.AddRazorPages().AddRazorPagesOptions(options =>
{
    options.Conventions.AuthorizeAreaFolder("Admin", "/");
});
```

> [!summary] Section Summary
> - File path under `/Pages/` directly determines the URL (convention-based routing)
> - `Index.cshtml` serves as the default document for its folder
> - No explicit route registration is needed -- the file system IS the routing table
> - The root folder can be customized, and areas are supported

---

## The @page Directive

The **`@page` directive** is what makes a `.cshtml` file a Razor Page. Without it, the file is just a regular Razor view (not routable).

```cshtml
@page
@model IndexModel

<h1>Products</h1>
```

> [!danger] Critical Requirement
> `@page` must be the **first Razor directive** in the file. If `@model` comes before `@page`, the file will not be treated as a Razor Page and will not be routable. You will get a confusing "page not found" error.

`@page` can also include **route template parameters**:

```cshtml
@page "{id:int}"
@* URL: /Products/Edit/42 *@

@page "{id:int?}"
@* URL: /Products/Edit or /Products/Edit/42 (optional parameter) *@

@page "{category}/{id:int}"
@* URL: /Products/electronics/42 *@
```

More on this in the [[#Page Routing and Route Parameters]] section.

> [!summary] Section Summary
> - `@page` is required and must be the first directive -- it makes the file a routable Razor Page
> - Without `@page`, the file is a regular view, not a page
> - `@page` can include route template parameters for dynamic URLs

---

## The PageModel Class

The **PageModel** class is the code-behind for a Razor Page. It is the equivalent of a controller action but scoped to a single page.

```csharp
// /Pages/Products/Index.cshtml.cs
public class IndexModel : PageModel
{
    private readonly IProductService _productService;

    public IndexModel(IProductService productService)
    {
        _productService = productService;
    }

    public List<Product> Products { get; set; } = new();

    public async Task OnGetAsync()
    {
        Products = await _productService.GetAllAsync();
    }
}
```

```cshtml
@* /Pages/Products/Index.cshtml *@
@page
@model MyApp.Pages.Products.IndexModel

<h1>Products</h1>
<table class="table">
    <thead>
        <tr>
            <th>Name</th>
            <th>Price</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var product in Model.Products)
        {
            <tr>
                <td>@product.Name</td>
                <td>@product.Price.ToString("C")</td>
                <td>
                    <a asp-page="./Edit" asp-route-id="@product.Id">Edit</a>
                    <a asp-page="./Delete" asp-route-id="@product.Id">Delete</a>
                </td>
            </tr>
        }
    </tbody>
</table>
<a asp-page="./Create" class="btn btn-primary">Create New Product</a>
```

**Key points:**
- The PageModel inherits from `Microsoft.AspNetCore.Mvc.RazorPages.PageModel`
- Constructor injection works just like controllers
- Public properties on the PageModel are accessible in the view via `Model.PropertyName`
- The `@model` directive in the `.cshtml` file points to the PageModel class (not a view model)

> [!warning] Common Misconception
> In MVC, `@model` refers to a view model (a data class). In Razor Pages, `@model` refers to the **PageModel class** (the code-behind). The PageModel is both the handler (like a controller) and the data provider (like a view model). Properties on the PageModel serve as the view model.

> [!summary] Section Summary
> - The PageModel class (`*.cshtml.cs`) is the code-behind that handles requests and holds page data
> - It supports constructor injection for services
> - Public properties on the PageModel are the view's data source (accessed via `Model.*`)
> - In Razor Pages, `@model` points to the PageModel, not a separate view model class

---

## Handler Methods

Handler methods in a PageModel respond to HTTP verbs. They follow the naming convention `On{Verb}[Async]`:

### OnGet / OnGetAsync

Handles GET requests -- typically loads data for display:

```csharp
public async Task OnGetAsync()
{
    Products = await _productService.GetAllAsync();
}
```

### OnPost / OnPostAsync

Handles POST requests -- typically processes form submissions:

```csharp
public async Task<IActionResult> OnPostAsync()
{
    if (!ModelState.IsValid)
    {
        return Page();  // Re-render the page with validation errors
    }

    await _productService.CreateAsync(Product);
    return RedirectToPage("./Index");
}
```

### Return Types

Handler methods can return:

| Return Type | Use Case |
|---|---|
| `void` / `Task` | Render the current page (implicit `Page()`) |
| `IActionResult` / `Task<IActionResult>` | Return `Page()`, `RedirectToPage()`, `NotFound()`, etc. |
| `PageResult` | Explicitly render the current page |

```csharp
// Implicit return -- renders the page
public async Task OnGetAsync()
{
    Products = await _productService.GetAllAsync();
}

// Explicit return -- can redirect, return 404, etc.
public async Task<IActionResult> OnGetAsync(int id)
{
    Product = await _productService.GetByIdAsync(id);
    if (Product == null)
    {
        return NotFound();
    }
    return Page();
}
```

> [!tip] Practical Tip
> Use `void`/`Task` return types for simple GET handlers that always render the page. Use `IActionResult`/`Task<IActionResult>` when the handler might redirect, return a 404, or return different results based on conditions.

> [!summary] Section Summary
> - Handler methods follow the `On{Verb}[Async]` naming convention
> - `OnGetAsync()` handles GET requests (data loading)
> - `OnPostAsync()` handles POST requests (form processing)
> - Return `Page()` to render, `RedirectToPage()` to navigate, `NotFound()` for 404s
> - `void`/`Task` return types implicitly call `Page()`

---

## Named Handlers

When a page needs multiple POST actions (e.g., an edit page with both "Save" and "Delete" buttons), use **named handlers**:

```csharp
// /Pages/Products/Edit.cshtml.cs
public class EditModel : PageModel
{
    // ...

    public async Task<IActionResult> OnPostSaveAsync()
    {
        if (!ModelState.IsValid)
        {
            return Page();
        }
        await _productService.UpdateAsync(Product);
        return RedirectToPage("./Index");
    }

    public async Task<IActionResult> OnPostDeleteAsync()
    {
        await _productService.DeleteAsync(Product.Id);
        return RedirectToPage("./Index");
    }

    public async Task<IActionResult> OnPostPublishAsync()
    {
        await _productService.PublishAsync(Product.Id);
        return RedirectToPage("./Edit", new { id = Product.Id });
    }
}
```

```cshtml
@* /Pages/Products/Edit.cshtml *@
@page "{id:int}"
@model EditModel

<form method="post">
    <input asp-for="Product.Id" type="hidden" />
    <div class="mb-3">
        <label asp-for="Product.Name"></label>
        <input asp-for="Product.Name" class="form-control" />
        <span asp-validation-for="Product.Name" class="text-danger"></span>
    </div>
    <div class="mb-3">
        <label asp-for="Product.Price"></label>
        <input asp-for="Product.Price" class="form-control" />
        <span asp-validation-for="Product.Price" class="text-danger"></span>
    </div>

    <button type="submit" asp-page-handler="Save" class="btn btn-primary">Save</button>
    <button type="submit" asp-page-handler="Publish" class="btn btn-success">Publish</button>
    <button type="submit" asp-page-handler="Delete" class="btn btn-danger"
            onclick="return confirm('Are you sure?')">
        Delete
    </button>
</form>
```

**How it works:**
- `asp-page-handler="Save"` adds `?handler=Save` to the form's action URL
- ASP.NET Core matches the `handler` query parameter to `OnPost**Save**Async()`
- The handler name is the part between `OnPost` and `Async`

> [!ad-note] URL Format
> Named handlers append `?handler=Name` to the URL by default. If you prefer clean URLs, you can configure route-based handler names:
> ```csharp
> builder.Services.AddRazorPages().AddRazorPagesOptions(options =>
> {
>     options.Conventions.ConfigureFilter(new AutoValidateAntiforgeryTokenAttribute());
> });
> ```
> Or use a route template: `@page "{id:int}/{handler?}"`

> [!summary] Section Summary
> - Named handlers allow multiple POST actions on a single page (`OnPostSaveAsync`, `OnPostDeleteAsync`)
> - `asp-page-handler="Name"` on buttons/forms routes to the corresponding handler
> - The handler name maps to the method name: `asp-page-handler="Save"` -> `OnPostSaveAsync()`
> - By default, handlers are passed via `?handler=Name` query parameter

---

## Model Binding with BindProperty

In Razor Pages, model binding uses the `[BindProperty]` attribute to bind form data to PageModel properties:

```csharp
public class CreateModel : PageModel
{
    private readonly IProductService _productService;

    public CreateModel(IProductService productService)
    {
        _productService = productService;
    }

    [BindProperty]
    public Product Product { get; set; }

    public void OnGet()
    {
        // Product is not bound on GET -- initialize defaults if needed
        Product = new Product { Price = 0 };
    }

    public async Task<IActionResult> OnPostAsync()
    {
        // Product is automatically populated from form data
        if (!ModelState.IsValid)
        {
            return Page();
        }

        await _productService.CreateAsync(Product);
        return RedirectToPage("./Index");
    }
}
```

### Key Behaviors

- `[BindProperty]` binds on **POST by default** (not GET)
- To bind on GET as well: `[BindProperty(SupportsGet = true)]`
- Multiple properties can be bound:

```csharp
[BindProperty]
public Product Product { get; set; }

[BindProperty]
public List<int> SelectedCategoryIds { get; set; }
```

### Binding All Properties

```csharp
[BindProperties]  // Note the plural -- binds ALL public properties
public class CreateModel : PageModel
{
    public Product Product { get; set; }
    public string Notes { get; set; }

    // Both are bound from form data on POST
}
```

> [!warning] Common Misconception
> Properties without `[BindProperty]` are NOT populated from form data. If you forget the attribute, the property will be `null` in `OnPostAsync()` even if the form sends the data. This is the most common Razor Pages debugging issue for beginners.

> [!tip] Practical Tip
> Be selective with `[BindProperty]`. Only bind properties that come from the form. Properties that hold display-only data (like dropdown lists) should NOT be bound -- they need to be repopulated in the handler if `Page()` is returned:
> ```csharp
> [BindProperty]
> public Product Product { get; set; }      // From form
>
> public SelectList Categories { get; set; } // Display only -- no [BindProperty]
>
> public async Task<IActionResult> OnPostAsync()
> {
>     if (!ModelState.IsValid)
>     {
>         Categories = await LoadCategories(); // Must repopulate!
>         return Page();
>     }
>     // ...
> }
> ```

> [!summary] Section Summary
> - `[BindProperty]` marks properties for automatic model binding from form data
> - By default, binding only occurs on POST; use `SupportsGet = true` for GET binding
> - `[BindProperties]` (plural) binds all public properties on the PageModel
> - Forgetting `[BindProperty]` results in null properties -- the most common Razor Pages mistake
> - Display-only properties (dropdowns, lists) must be repopulated when returning `Page()`

---

## Page Routing and Route Parameters

The `@page` directive can include a route template for dynamic URL segments:

```cshtml
@* Simple required parameter *@
@page "{id:int}"
@* URL: /Products/Details/42 *@

@* Optional parameter *@
@page "{id:int?}"
@* URL: /Products/Details or /Products/Details/42 *@

@* Multiple parameters *@
@page "{category}/{id:int}"
@* URL: /Products/electronics/42 *@

@* Catch-all parameter *@
@page "{*slug}"
@* URL: /Blog/2024/01/my-post-title (slug = "2024/01/my-post-title") *@
```

### Accessing Route Parameters in the PageModel

Route parameters are passed as method parameters to handlers:

```csharp
public class DetailsModel : PageModel
{
    // Option 1: Handler method parameter
    public async Task OnGetAsync(int id)
    {
        Product = await _productService.GetByIdAsync(id);
    }

    // Option 2: [BindProperty] with SupportsGet
    [BindProperty(SupportsGet = true)]
    public int Id { get; set; }

    public async Task OnGetAsync()
    {
        Product = await _productService.GetByIdAsync(Id);
    }
}
```

### Route Constraints

Standard ASP.NET Core [[17.05 - Routing|route constraints]] work in Razor Pages:

| Constraint | Example | Matches |
|---|---|---|
| `int` | `{id:int}` | `42`, `1` |
| `bool` | `{active:bool}` | `true`, `false` |
| `datetime` | `{date:datetime}` | `2024-01-15` |
| `guid` | `{id:guid}` | `CD2C1638-...` |
| `minlength(n)` | `{slug:minlength(3)}` | Strings with 3+ chars |
| `regex(expr)` | `{code:regex(^[A-Z]{{3}}$)}` | Three uppercase letters |

### Overriding the Route Entirely

```cshtml
@page "/custom/url/path/{id:int}"
@* This page is now at /custom/url/path/42, not its file-system path *@
```

> [!summary] Section Summary
> - `@page "{id:int}"` adds route parameters to the page's URL
> - Parameters can be optional (`?`), constrained (`int`, `guid`, `minlength`), or catch-all (`*`)
> - Route parameters are accessible as handler method parameters or `[BindProperty(SupportsGet = true)]`
> - The entire route can be overridden: `@page "/custom/path/{id}"`

---

## Navigation with RedirectToPage

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

## Layouts, Partials, and View Components

Razor Pages use the same [[Layouts and Sections|layout system]], [[Partial Views and View Components|partial views and view components]], and [[Tag Helpers]] as MVC views.

### _ViewStart.cshtml for Razor Pages

```cshtml
@* /Pages/_ViewStart.cshtml *@
@{
    Layout = "_Layout";
}
```

Razor Pages search for layouts in:
1. The same folder as the page
2. `/Pages/Shared/`
3. `/Views/Shared/` (shared with MVC views)

### _ViewImports.cshtml for Razor Pages

```cshtml
@* /Pages/_ViewImports.cshtml *@
@using MyApp
@using MyApp.Models
@namespace MyApp.Pages
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

> [!ad-note] Shared Layouts Between MVC and Razor Pages
> If your application uses both MVC controllers and Razor Pages, a layout in `/Views/Shared/` is accessible to both. You do not need duplicate layouts.

### Partial Views

```cshtml
@* Works identically to MVC *@
<partial name="_ProductCard" model="product" />
```

### View Components

```cshtml
@* Works identically to MVC *@
<vc:cart-summary />
@await Component.InvokeAsync("RecentProducts", new { count = 5 })
```

> [!summary] Section Summary
> - Razor Pages use the same `_ViewStart.cshtml`, `_ViewImports.cshtml`, and layout system as MVC
> - Layouts are searched in the page's folder, `/Pages/Shared/`, and `/Views/Shared/`
> - Partial views and view components work identically to MVC
> - MVC and Razor Pages can share layouts, partials, and view components

---

## Advantages of Razor Pages

1. **Simpler for CRUD**: a Create page is two files (`Create.cshtml` + `Create.cshtml.cs`), not three (controller + view + view model)
2. **Self-contained**: the page and its logic live together, making the feature easy to find and understand
3. **Convention-based routing**: file path = URL, no route configuration needed
4. **Less boilerplate**: no controller class with multiple actions, no `[HttpGet]`/`[HttpPost]` attributes
5. **Easier for beginners**: the programming model is more intuitive than MVC's indirection
6. **Full MVC power**: still has model binding, validation, filters, DI, tag helpers, etc.
7. **Coexists with MVC**: use Pages for some features, controllers for others

> [!tip] Practical Tip
> The ASP.NET Core project templates use Razor Pages by default for Identity UI (login, register, manage account). Even in MVC-heavy applications, the Identity pages are Razor Pages under `/Areas/Identity/Pages/`.

> [!summary] Section Summary
> - Razor Pages reduces file count and ceremony for page-oriented features
> - Convention-based routing eliminates manual route configuration
> - The self-contained nature (page + model together) improves feature discoverability
> - Full access to MVC infrastructure: model binding, validation, DI, filters, tag helpers

---

## Limitations of Razor Pages

1. **Complex pages get messy**: a page with many handlers and properties can become a "god class"
2. **Testing isolation**: while PageModel is testable, the tight coupling of handlers to page rendering makes some tests less clean than controller tests
3. **Not great for APIs**: Razor Pages are designed for HTML rendering, not JSON APIs (use controllers or Minimal APIs)
4. **No multi-view support**: each page renders exactly one view, unlike a controller that can return different views from different actions
5. **Large teams**: the file-per-page model can lead to very large `/Pages/` directories in big applications
6. **Complex workflows**: multi-step wizards or approval flows with branching logic are easier to model with controllers

> [!warning] Common Misconception
> Razor Pages is not a "simplified" or "lesser" version of MVC. It is a **different programming model** for a different class of problems. Choosing Razor Pages for a CRUD admin panel is not "dumbing down" the architecture -- it is using the right tool for the job.

> [!summary] Section Summary
> - Razor Pages can become unwieldy for pages with many handlers ("god PageModel")
> - Not suitable for APIs -- use controllers or Minimal APIs
> - Single view per page (no multi-view support like controllers)
> - Complex multi-step workflows are better served by MVC controllers
> - Razor Pages is a valid architectural choice, not a simplified alternative

---

## Real-World Example: Product CRUD Pages

A complete set of CRUD pages for managing products.

### Index Page (List)

```csharp
// /Pages/Products/Index.cshtml.cs
public class IndexModel : PageModel
{
    private readonly IProductService _productService;

    public IndexModel(IProductService productService)
    {
        _productService = productService;
    }

    public List<Product> Products { get; set; } = new();

    [BindProperty(SupportsGet = true)]
    public string SearchTerm { get; set; }

    public async Task OnGetAsync()
    {
        Products = string.IsNullOrWhiteSpace(SearchTerm)
            ? await _productService.GetAllAsync()
            : await _productService.SearchAsync(SearchTerm);
    }
}
```

```cshtml
@* /Pages/Products/Index.cshtml *@
@page
@model MyApp.Pages.Products.IndexModel

<h1>Products</h1>

<form method="get" class="mb-3">
    <div class="input-group">
        <input asp-for="SearchTerm" class="form-control" placeholder="Search products..." />
        <button type="submit" class="btn btn-outline-secondary">Search</button>
    </div>
</form>

<a asp-page="./Create" class="btn btn-primary mb-3">Create New Product</a>

@if (!Model.Products.Any())
{
    <p class="text-muted">No products found.</p>
}
else
{
    <table class="table table-striped">
        <thead>
            <tr>
                <th>Name</th>
                <th>Category</th>
                <th>Price</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var product in Model.Products)
            {
                <tr>
                    <td>@product.Name</td>
                    <td>@product.CategoryName</td>
                    <td>@product.Price.ToString("C")</td>
                    <td>
                        <a asp-page="./Edit" asp-route-id="@product.Id"
                           class="btn btn-sm btn-outline-primary">Edit</a>
                        <a asp-page="./Delete" asp-route-id="@product.Id"
                           class="btn btn-sm btn-outline-danger">Delete</a>
                    </td>
                </tr>
            }
        </tbody>
    </table>
}
```

### Create Page

```csharp
// /Pages/Products/Create.cshtml.cs
public class CreateModel : PageModel
{
    private readonly IProductService _productService;
    private readonly ICategoryService _categoryService;

    public CreateModel(IProductService productService, ICategoryService categoryService)
    {
        _productService = productService;
        _categoryService = categoryService;
    }

    [BindProperty]
    public ProductCreateDto Product { get; set; }

    public SelectList Categories { get; set; }

    public async Task OnGetAsync()
    {
        Categories = new SelectList(
            await _categoryService.GetAllAsync(), "Id", "Name");
    }

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid)
        {
            // Must repopulate non-bound properties
            Categories = new SelectList(
                await _categoryService.GetAllAsync(), "Id", "Name");
            return Page();
        }

        await _productService.CreateAsync(Product);
        TempData["SuccessMessage"] = "Product created successfully.";
        return RedirectToPage("./Index");
    }
}
```

```cshtml
@* /Pages/Products/Create.cshtml *@
@page
@model MyApp.Pages.Products.CreateModel

<h1>Create Product</h1>

<form method="post">
    <div asp-validation-summary="All" class="text-danger"></div>

    <div class="mb-3">
        <label asp-for="Product.Name"></label>
        <input asp-for="Product.Name" class="form-control" />
        <span asp-validation-for="Product.Name" class="text-danger"></span>
    </div>

    <div class="mb-3">
        <label asp-for="Product.Description"></label>
        <textarea asp-for="Product.Description" class="form-control" rows="4"></textarea>
        <span asp-validation-for="Product.Description" class="text-danger"></span>
    </div>

    <div class="mb-3">
        <label asp-for="Product.Price"></label>
        <input asp-for="Product.Price" class="form-control" />
        <span asp-validation-for="Product.Price" class="text-danger"></span>
    </div>

    <div class="mb-3">
        <label asp-for="Product.CategoryId"></label>
        <select asp-for="Product.CategoryId" asp-items="Model.Categories" class="form-select">
            <option value="">-- Select Category --</option>
        </select>
        <span asp-validation-for="Product.CategoryId" class="text-danger"></span>
    </div>

    <button type="submit" class="btn btn-primary">Create</button>
    <a asp-page="./Index" class="btn btn-secondary">Cancel</a>
</form>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

### Edit Page

```csharp
// /Pages/Products/Edit.cshtml.cs
public class EditModel : PageModel
{
    private readonly IProductService _productService;
    private readonly ICategoryService _categoryService;

    public EditModel(IProductService productService, ICategoryService categoryService)
    {
        _productService = productService;
        _categoryService = categoryService;
    }

    [BindProperty]
    public ProductEditDto Product { get; set; }

    public SelectList Categories { get; set; }

    public async Task<IActionResult> OnGetAsync(int id)
    {
        var product = await _productService.GetByIdAsync(id);
        if (product == null)
        {
            return NotFound();
        }

        Product = new ProductEditDto
        {
            Id = product.Id,
            Name = product.Name,
            Description = product.Description,
            Price = product.Price,
            CategoryId = product.CategoryId
        };

        Categories = new SelectList(
            await _categoryService.GetAllAsync(), "Id", "Name");

        return Page();
    }

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid)
        {
            Categories = new SelectList(
                await _categoryService.GetAllAsync(), "Id", "Name");
            return Page();
        }

        var success = await _productService.UpdateAsync(Product);
        if (!success)
        {
            return NotFound();
        }

        TempData["SuccessMessage"] = "Product updated successfully.";
        return RedirectToPage("./Index");
    }
}
```

### Delete Page (Confirmation)

```csharp
// /Pages/Products/Delete.cshtml.cs
public class DeleteModel : PageModel
{
    private readonly IProductService _productService;

    public DeleteModel(IProductService productService)
    {
        _productService = productService;
    }

    public Product Product { get; set; }

    public async Task<IActionResult> OnGetAsync(int id)
    {
        Product = await _productService.GetByIdAsync(id);
        if (Product == null)
        {
            return NotFound();
        }
        return Page();
    }

    public async Task<IActionResult> OnPostAsync(int id)
    {
        var success = await _productService.DeleteAsync(id);
        if (!success)
        {
            return NotFound();
        }

        TempData["SuccessMessage"] = "Product deleted.";
        return RedirectToPage("./Index");
    }
}
```

```cshtml
@* /Pages/Products/Delete.cshtml *@
@page "{id:int}"
@model MyApp.Pages.Products.DeleteModel

<h1>Delete Product</h1>

<div class="alert alert-danger">
    <h4 class="alert-heading">Are you sure?</h4>
    <p>You are about to permanently delete the following product:</p>
</div>

<dl class="row">
    <dt class="col-sm-3">Name</dt>
    <dd class="col-sm-9">@Model.Product.Name</dd>

    <dt class="col-sm-3">Category</dt>
    <dd class="col-sm-9">@Model.Product.CategoryName</dd>

    <dt class="col-sm-3">Price</dt>
    <dd class="col-sm-9">@Model.Product.Price.ToString("C")</dd>
</dl>

<form method="post" asp-route-id="@Model.Product.Id">
    <button type="submit" class="btn btn-danger">Confirm Delete</button>
    <a asp-page="./Index" class="btn btn-secondary">Cancel</a>
</form>
```

This CRUD set demonstrates:
- Handler methods for GET and POST
- `[BindProperty]` for form model binding
- Route parameters (`{id:int}`)
- `RedirectToPage()` for post-redirect-get (PRG) pattern
- Validation with `ModelState.IsValid` and `Page()` for re-rendering
- Repopulating non-bound properties (Categories) when returning `Page()`
- [[Tag Helpers]] for forms, links, and validation
- `TempData` for success messages across redirects

> [!summary] Section Summary
> - A complete CRUD set consists of Index, Create, Edit, and Delete pages
> - Each page follows the PageModel pattern with GET (display) and POST (process) handlers
> - The post-redirect-get (PRG) pattern prevents duplicate form submissions
> - Non-bound display data must be repopulated when returning `Page()` after validation failure
> - `TempData` carries success messages across the redirect

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Razor Pages** is a page-focused programming model in ASP.NET Core where each page is a self-contained pair: a `.cshtml` Razor view and a `.cshtml.cs` PageModel code-behind. Pages live in the `/Pages/` directory and use **convention-based routing** where the file path maps directly to the URL (`/Pages/Products/Index.cshtml` -> `/Products`).
>
> The **`@page` directive** (required, must be first) makes a `.cshtml` file a routable Razor Page. The **PageModel** class inherits from `PageModel`, uses constructor injection for services, and exposes public properties that serve as the view model. **Handler methods** follow the `On{Verb}[Async]` pattern: `OnGetAsync()` loads data, `OnPostAsync()` processes forms. **Named handlers** (`OnPostDeleteAsync()`) support multiple POST actions on a single page, triggered via `asp-page-handler`.
>
> **`[BindProperty]`** marks properties for automatic model binding from form data (POST by default; `SupportsGet = true` for GET). This is the most critical attribute in Razor Pages -- forgetting it results in null properties. **Route parameters** are defined in the `@page` directive (`@page "{id:int}"`) and support all standard constraints.
>
> Razor Pages shares the same [[Layouts and Sections|layout system]], [[Partial Views and View Components|partial views and view components]], [[Tag Helpers]], and validation infrastructure as MVC. The two models coexist naturally in the same application. Razor Pages excels at **form-heavy CRUD pages** and **admin panels**, while MVC controllers are better suited for **complex routing, APIs, and multi-view workflows**. The choice is per-feature, not per-application.

---

## Related Topics

- [[Razor Syntax]] -- the markup syntax used in Razor Pages' `.cshtml` files
- [[Layouts and Sections]] -- Razor Pages use the same layout system as MVC
- [[Partial Views and View Components]] -- reusable fragments work identically in Razor Pages
- [[Tag Helpers]] -- `asp-page`, `asp-route-{param}`, `asp-page-handler` for Razor Pages navigation
- [[17.05 - Routing]] -- Razor Pages convention-based routing is built on the core routing system
- [[17.06 - Controllers and Actions]] -- the MVC alternative to Razor Pages
- [[17.03 - Dependency Injection]] -- constructor injection in PageModel classes
