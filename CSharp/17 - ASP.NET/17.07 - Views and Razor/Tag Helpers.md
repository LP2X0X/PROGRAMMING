---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---

# Tag Helpers

> [!ad-note] Overview
> Tag helpers are server-side components that participate in rendering HTML elements in Razor files. They look like standard HTML attributes (e.g., `asp-controller`, `asp-for`) but execute C# code on the server to generate correct URLs, form fields, validation attributes, and more. They replace the older `@Html.*` helpers with a more natural, HTML-like syntax that designers and front-end developers can work with.

## Table of Contents

- [What Tag Helpers Are](#what-tag-helpers-are)
- [Enabling Tag Helpers](#enabling-tag-helpers)
- [Anchor Tag Helper](#anchor-tag-helper)
- [Form Tag Helper](#form-tag-helper)
- [Input Tag Helper](#input-tag-helper)
- [Label, Select, and Textarea Tag Helpers](#label-select-and-textarea-tag-helpers)
- [Validation Tag Helpers](#validation-tag-helpers)
- [Image, Link, and Script Tag Helpers](#image-link-and-script-tag-helpers)
- [Environment Tag Helper](#environment-tag-helper)
- [Partial Tag Helper](#partial-tag-helper)
- [Cache Tag Helpers](#cache-tag-helpers)
- [Tag Helpers vs HTML Helpers](#tag-helpers-vs-html-helpers)
- [asp-for Deep Dive](#asp-for-deep-dive)
- [Custom Tag Helpers](#custom-tag-helpers)
- [Real-World Example: Bootstrap Alert Tag Helper](#real-world-example-bootstrap-alert-tag-helper)
- [Comprehensive Summary](#comprehensive-summary)
- [Related Topics](#related-topics)

---

## What Tag Helpers Are

**Tag helpers** are C# classes that attach to HTML elements and modify their rendering on the server side. When Razor encounters an HTML element with tag helper attributes (like `asp-controller`), it invokes the corresponding tag helper class, which can:

- Modify the element's attributes (add `href`, `action`, `name`, `id`, etc.)
- Add new child content
- Suppress the element entirely
- Replace the element with different markup

The key insight is that tag helpers **look like HTML**. Compare:

```cshtml
@* HTML Helper (old way) *@
@Html.ActionLink("Products", "Index", "Products", null, new { @class = "nav-link" })

@* Tag Helper (modern way) *@
<a asp-controller="Products" asp-action="Index" class="nav-link">Products</a>
```

The tag helper version is valid HTML that any designer or browser developer tool can parse. The `asp-*` attributes are processed on the server and removed from the final HTML output.

> [!summary] Section Summary
> - Tag helpers are server-side C# classes that enhance HTML elements
> - They use `asp-*` attributes that look like regular HTML attributes
> - They replace the older `@Html.*` helper methods with a more natural, HTML-like syntax
> - The `asp-*` attributes are processed on the server and stripped from the rendered output

---

## Enabling Tag Helpers

Tag helpers must be registered in `_ViewImports.cshtml` using the `@addTagHelper` directive:

```cshtml
@* /Views/_ViewImports.cshtml *@

@* Enable all built-in ASP.NET Core tag helpers *@
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

@* Enable custom tag helpers from your application *@
@addTagHelper *, MyApp

@* Enable a specific tag helper only *@
@addTagHelper MyApp.TagHelpers.AlertTagHelper, MyApp
```

### Disabling Tag Helpers

```cshtml
@* Remove a specific tag helper *@
@removeTagHelper Microsoft.AspNetCore.Mvc.TagHelpers.EnvironmentTagHelper, Microsoft.AspNetCore.Mvc.TagHelpers

@* Opt out for a single element using ! prefix *@
<!a asp-controller="Home">This will NOT be processed</!a>
```

### Tag Helper Prefix

To make it explicit which elements are tag helpers (useful in large teams):

```cshtml
@tagHelperPrefix th:

@* Now you must write: *@
<th:a asp-controller="Home" asp-action="Index">Home</th:a>
```

> [!tip] Practical Tip
> Most projects simply use `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` and their own assembly wildcard. The prefix approach is rarely used outside of very large projects with mixed teams.

> [!summary] Section Summary
> - `@addTagHelper` in `_ViewImports.cshtml` enables tag helpers for all views in that directory tree
> - Use wildcard (`*`) to enable all tag helpers from an assembly
> - `@removeTagHelper` disables specific tag helpers; `!` prefix opts out individual elements
> - `@tagHelperPrefix` requires a prefix on all tag helper elements (rarely used)

---

## Anchor Tag Helper

The **anchor tag helper** generates correct URLs from routing information, eliminating hardcoded paths:

```cshtml
@* Basic usage *@
<a asp-controller="Products" asp-action="Index">All Products</a>
@* Renders: <a href="/Products">All Products</a> *@

@* With route parameter *@
<a asp-controller="Products" asp-action="Details" asp-route-id="42">
    Widget Pro
</a>
@* Renders: <a href="/Products/Details/42">Widget Pro</a> *@

@* Multiple route parameters *@
<a asp-controller="Products" asp-action="ByCategory"
   asp-route-category="electronics"
   asp-route-page="2">
    Electronics - Page 2
</a>
@* Renders: <a href="/Products/ByCategory/electronics?page=2">Electronics - Page 2</a> *@

@* Using a named route *@
<a asp-route="product-detail" asp-route-id="42">Widget Pro</a>

@* Linking to an area *@
<a asp-area="Admin" asp-controller="Dashboard" asp-action="Index">Admin</a>

@* Linking to a Razor Page *@
<a asp-page="/Products/Details" asp-route-id="42">Widget Pro</a>

@* Fragment (hash) *@
<a asp-controller="Products" asp-action="Details"
   asp-route-id="42" asp-fragment="reviews">
    See Reviews
</a>
@* Renders: <a href="/Products/Details/42#reviews">See Reviews</a> *@
```

**Available attributes:**

| Attribute | Purpose |
|---|---|
| `asp-controller` | Target controller name |
| `asp-action` | Target action method name |
| `asp-route-{name}` | Route parameter value (any name after the dash) |
| `asp-route` | Named route |
| `asp-area` | MVC area name |
| `asp-page` | Target Razor Page |
| `asp-page-handler` | Razor Page handler name |
| `asp-protocol` | Force `https` or `http` |
| `asp-host` | Override the hostname |
| `asp-fragment` | URL fragment (`#section`) |

> [!warning] Common Misconception
> `asp-route-id` is not a special attribute. The `id` part is just the route parameter name. You can use `asp-route-anything` to pass any route value: `asp-route-category`, `asp-route-slug`, `asp-route-year`, etc.

> [!summary] Section Summary
> - The anchor tag helper replaces `@Html.ActionLink()` with natural HTML anchor elements
> - `asp-controller`, `asp-action`, and `asp-route-{param}` generate correct URLs from routing
> - Works with areas, Razor Pages, named routes, fragments, and protocol overrides
> - `asp-route-{name}` is a dynamic attribute where `{name}` is any route parameter

---

## Form Tag Helper

The **form tag helper** generates the `action` URL and includes anti-forgery tokens automatically:

```cshtml
<form asp-controller="Products" asp-action="Create" method="post">
    @* Form content *@
    <button type="submit">Create Product</button>
</form>
```

Renders:

```html
<form action="/Products/Create" method="post">
    <input name="__RequestVerificationToken" type="hidden" value="CfDJ8..." />
    <!-- Form content -->
    <button type="submit">Create Product</button>
</form>
```

> [!ad-note] Anti-Forgery Token
> The form tag helper automatically inserts a hidden `__RequestVerificationToken` field for POST forms. This works with the `[ValidateAntiForgeryToken]` attribute on controller actions (or the global `AutoValidateAntiforgeryToken` filter) to prevent CSRF attacks. Set `asp-antiforgery="false"` to disable this behavior.

```cshtml
@* Razor Page form *@
<form asp-page="/Products/Create" method="post">
    @* content *@
</form>

@* Form with named handler *@
<form asp-page="/Products/Edit" asp-page-handler="Delete" method="post">
    <button type="submit">Delete Product</button>
</form>

@* Disable anti-forgery *@
<form asp-controller="Search" asp-action="Query" method="get" asp-antiforgery="false">
    <input type="text" name="q" />
    <button type="submit">Search</button>
</form>
```

> [!summary] Section Summary
> - The form tag helper generates the `action` attribute from routing parameters
> - It automatically includes an anti-forgery token for POST forms (CSRF protection)
> - Works with both MVC controllers and [[Razor Pages]] handlers
> - `asp-antiforgery="false"` disables the automatic token (appropriate for GET forms)

---

## Input Tag Helper

The **input tag helper** is arguably the most powerful built-in tag helper. It uses `asp-for` to generate the `name`, `id`, `type`, and `data-val-*` validation attributes from the model property and its data annotations.

```cshtml
@model ProductViewModel

<input asp-for="Name" class="form-control" />
```

Given this model:

```csharp
public class ProductViewModel
{
    [Required(ErrorMessage = "Product name is required")]
    [StringLength(100, MinimumLength = 3)]
    [Display(Name = "Product Name")]
    public string Name { get; set; }
}
```

Renders:

```html
<input class="form-control"
       type="text"
       id="Name"
       name="Name"
       value=""
       data-val="true"
       data-val-required="Product name is required"
       data-val-length="The field Product Name must be a string with a minimum length of 3 and a maximum length of 100."
       data-val-length-max="100"
       data-val-length-min="3" />
```

### Type Inference

The tag helper infers the HTML `type` attribute from the C# property type:

| C# Type / Annotation | HTML type |
|---|---|
| `string` | `text` |
| `int`, `long`, `decimal`, `double` | `number` |
| `bool` | `checkbox` |
| `DateTime` | `datetime-local` |
| `[DataType(DataType.Password)]` | `password` |
| `[DataType(DataType.EmailAddress)]` | `email` |
| `[DataType(DataType.Url)]` | `url` |
| `[DataType(DataType.PhoneNumber)]` | `tel` |
| `[DataType(DataType.Date)]` | `date` |
| `[DataType(DataType.Time)]` | `time` |
| `byte[]` | `file` |
| `[HiddenInput]` | `hidden` |

> [!tip] Practical Tip
> Always apply `[DataType]` annotations on your model for proper input types. A `string Email` property renders as `type="text"` unless you annotate it with `[DataType(DataType.EmailAddress)]` or `[EmailAddress]`, which gives you `type="email"` and mobile-friendly keyboards.

> [!summary] Section Summary
> - `asp-for` binds an input to a model property, generating `name`, `id`, `type`, and validation attributes
> - Data annotations on the model property drive both the HTML type and client-side validation
> - Type inference maps C# types and `[DataType]` annotations to the correct HTML `type`
> - The generated `data-val-*` attributes enable jQuery Unobtrusive Validation

---

## Label, Select, and Textarea Tag Helpers

### Label Tag Helper

```cshtml
<label asp-for="Name"></label>
@* Renders: <label for="Name">Product Name</label> *@
@* The text comes from [Display(Name = "...")] or the property name *@
```

### Select Tag Helper

```cshtml
<select asp-for="CategoryId" asp-items="Model.Categories" class="form-select">
    <option value="">-- Select Category --</option>
</select>
```

Where `Model.Categories` is a `SelectList` or `IEnumerable<SelectListItem>`:

```csharp
public class ProductViewModel
{
    public int CategoryId { get; set; }
    public SelectList Categories { get; set; }
}

// In the controller:
var viewModel = new ProductViewModel
{
    Categories = new SelectList(
        await _context.Categories.ToListAsync(),
        "Id",       // value field
        "Name"      // text field
    )
};
```

For enum-based selects:

```cshtml
<select asp-for="Status" asp-items="Html.GetEnumSelectList<OrderStatus>()" class="form-select">
</select>
```

### Textarea Tag Helper

```cshtml
<textarea asp-for="Description" class="form-control" rows="5"></textarea>
@* Generates name, id, and validation attributes just like the input tag helper *@
```

> [!summary] Section Summary
> - `<label asp-for>` generates `for` attribute and display text from model metadata
> - `<select asp-for asp-items>` generates a dropdown from a `SelectList` or `IEnumerable<SelectListItem>`
> - `Html.GetEnumSelectList<T>()` creates select items from an enum type
> - `<textarea asp-for>` generates name, id, and validation attributes from the model property

---

## Validation Tag Helpers

Two tag helpers display validation messages generated by model binding and data annotations.

### Validation Message (Per-Field)

```cshtml
<div class="mb-3">
    <label asp-for="Name"></label>
    <input asp-for="Name" class="form-control" />
    <span asp-validation-for="Name" class="text-danger"></span>
</div>
```

Renders (when validation fails):

```html
<span class="text-danger field-validation-error" data-valmsg-for="Name" data-valmsg-replace="true">
    Product name is required
</span>
```

### Validation Summary (All Errors)

```cshtml
<div asp-validation-summary="All" class="text-danger"></div>
```

The `asp-validation-summary` attribute accepts:
- `All` -- shows all validation errors (property-level + model-level)
- `ModelOnly` -- shows only model-level errors (added via `ModelState.AddModelError("", "message")`)
- `None` -- disables the summary

### Client-Side Validation Scripts

To enable client-side validation, include the jQuery Validation and Unobtrusive Validation scripts. Typically done in a [[Layouts and Sections|section]]:

```cshtml
@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

Where `_ValidationScriptsPartial.cshtml` contains:

```cshtml
<script src="~/lib/jquery-validation/dist/jquery.validate.min.js"></script>
<script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"></script>
```

> [!ad-note] How Client-Side Validation Works
> The input tag helper generates `data-val-*` attributes from data annotations. jQuery Unobtrusive Validation reads these attributes and sets up client-side validation rules automatically. No custom JavaScript is needed for standard validations. This is why `asp-for` is so powerful -- it bridges C# annotations to both server-side and client-side validation.

> [!summary] Section Summary
> - `<span asp-validation-for="Property">` shows per-field validation messages
> - `<div asp-validation-summary="All">` shows all errors in a summary block
> - Client-side validation requires jQuery Validation + Unobtrusive Validation scripts
> - The `data-val-*` attributes generated by `asp-for` drive client-side validation automatically

---

## Image, Link, and Script Tag Helpers

These tag helpers add **cache-busting** via file version hashing.

### Image Tag Helper

```cshtml
<img src="~/images/logo.png" asp-append-version="true" />
```

Renders:

```html
<img src="/images/logo.png?v=Kl_dqr9NTwpYS27JkWm3Eg" />
```

The `v` query parameter is a hash of the file contents. When the file changes, the hash changes, forcing browsers to download the updated file instead of serving a stale cached version.

### Link and Script Tag Helpers

```cshtml
<link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
<script src="~/js/site.js" asp-append-version="true"></script>
```

### Script Tag Helper with Fallback

```cshtml
<script src="https://cdn.example.com/lib/jquery.min.js"
        asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
        asp-fallback-test="window.jQuery">
</script>
```

If the CDN fails, the fallback local script loads automatically.

> [!tip] Practical Tip
> Always use `asp-append-version="true"` on your CSS and JS references in production layouts. It eliminates the most common class of "I deployed but users see the old version" bugs with zero effort.

> [!summary] Section Summary
> - `asp-append-version="true"` adds a content-based hash query parameter for cache-busting
> - Works on `<img>`, `<link>`, and `<script>` elements
> - The hash changes when the file changes, forcing browser cache invalidation
> - `asp-fallback-src` and `asp-fallback-test` provide CDN fallback behavior for scripts

---

## Environment Tag Helper

The **environment tag helper** conditionally renders content based on the current hosting environment:

```cshtml
<environment include="Development">
    <link rel="stylesheet" href="~/css/site.css" />
    <script src="~/lib/jquery/dist/jquery.js"></script>
</environment>

<environment exclude="Development">
    <link rel="stylesheet"
          href="https://cdn.example.com/css/site.min.css"
          asp-fallback-href="~/css/site.min.css"
          asp-fallback-test-class="sr-only"
          asp-fallback-test-property="position"
          asp-fallback-test-value="absolute" />
    <script src="https://cdn.example.com/lib/jquery.min.js"
            asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
            asp-fallback-test="window.jQuery">
    </script>
</environment>

<environment include="Staging,Production">
    <script>
        // Analytics tracking code
    </script>
</environment>
```

The `include` attribute accepts a comma-separated list of environment names. The content is only rendered when the current `ASPNETCORE_ENVIRONMENT` matches.

> [!summary] Section Summary
> - `<environment include="Development">` renders content only in specified environments
> - `<environment exclude="Production">` renders content in all environments except the specified ones
> - Common use: unminified assets in Development, CDN + minified assets in Production
> - Environment is determined by the `ASPNETCORE_ENVIRONMENT` variable

---

## Partial Tag Helper

The partial tag helper renders [[Partial Views and View Components|partial views]] inline:

```cshtml
<partial name="_ProductCard" model="product" />

@* With additional ViewData *@
<partial name="_ProductCard"
         model="product"
         view-data="@(new ViewDataDictionary(ViewData) { { "ShowPrice", true } })" />

@* Specifying the full path *@
<partial name="/Views/Shared/_Header.cshtml" />
```

> [!ad-note] Partial vs View Component Tag Helpers
> Do not confuse `<partial name="...">` (renders a partial view) with `<vc:component-name>` (invokes a view component). Partials receive their data from the calling view; view components fetch their own data.

> [!summary] Section Summary
> - `<partial name="..." model="..." />` renders a partial view inline
> - Optional `view-data` attribute passes additional `ViewDataDictionary` entries
> - Discovery follows the same rules as other partial rendering methods

---

## Cache Tag Helpers

### In-Memory Cache Tag Helper

```cshtml
<cache expires-after="@TimeSpan.FromMinutes(10)">
    @* Expensive content rendered once, cached for 10 minutes *@
    @await Component.InvokeAsync("PopularProducts")
</cache>

<cache expires-sliding="@TimeSpan.FromMinutes(5)"
       vary-by-user="true"
       vary-by-query="category,page">
    @* Cached per user, varied by query string parameters *@
    <partial name="_RecommendedProducts" model="Model.Recommendations" />
</cache>
```

### Distributed Cache Tag Helper

For multi-server deployments:

```cshtml
<distributed-cache name="popular-products"
                   expires-after="@TimeSpan.FromMinutes(10)">
    @await Component.InvokeAsync("PopularProducts")
</distributed-cache>
```

Requires a distributed cache provider (Redis, SQL Server, etc.) registered in DI.

**Cache variation attributes:**

| Attribute | Varies cache by |
|---|---|
| `vary-by-user` | Authenticated user identity |
| `vary-by-query` | Query string parameters |
| `vary-by-route` | Route data values |
| `vary-by-cookie` | Cookie values |
| `vary-by-header` | Request headers |
| `vary-by` | Arbitrary string expression |

> [!summary] Section Summary
> - `<cache>` caches rendered HTML in memory with expiration policies
> - `<distributed-cache>` uses a distributed cache provider for multi-server scenarios
> - `vary-by-*` attributes create separate cache entries per user, query, route, etc.
> - Useful for expensive view components and partials that do not change frequently

---

## Tag Helpers vs HTML Helpers

| Aspect | Tag Helpers | HTML Helpers |
|---|---|---|
| **Syntax** | HTML-like (`<a asp-action>`) | C# method calls (`@Html.ActionLink()`) |
| **Readability** | Looks like standard HTML | Breaks the HTML flow |
| **Designer-friendly** | Yes -- valid HTML attributes | No -- requires C# knowledge |
| **IntelliSense** | Full support in Visual Studio | Full support |
| **Custom creation** | Inherit from `TagHelper` | Write extension methods on `IHtmlHelper` |
| **Nesting** | Natural HTML nesting | Clunky lambda expressions |
| **CSS class** | `class="..."` (normal HTML) | `new { @class = "..." }` (anonymous object) |

```cshtml
@* HTML Helper *@
@Html.ActionLink("Products", "Index", "Products", null, new { @class = "nav-link" })

@* Equivalent Tag Helper *@
<a asp-controller="Products" asp-action="Index" class="nav-link">Products</a>
```

```cshtml
@* HTML Helper for a form field *@
@Html.TextBoxFor(m => m.Name, new { @class = "form-control", placeholder = "Enter name" })

@* Equivalent Tag Helper *@
<input asp-for="Name" class="form-control" placeholder="Enter name" />
```

> [!ad-note] Migration Note
> HTML helpers still work in ASP.NET Core and are not deprecated. However, tag helpers are the recommended approach for new development. You can mix both in the same view during a gradual migration.

> [!summary] Section Summary
> - Tag helpers provide an HTML-native syntax that is more readable and designer-friendly
> - HTML helpers use C# method calls that break the HTML flow
> - Tag helpers are the recommended approach for new projects
> - Both coexist in the same view -- no need for a full migration

---

## asp-for Deep Dive

The `asp-for` attribute is the backbone of tag helpers for form elements. Understanding what it generates is essential for working with forms.

### What asp-for Generates

Given a property:

```csharp
public class OrderViewModel
{
    [Required]
    [Display(Name = "Customer Email")]
    [DataType(DataType.EmailAddress)]
    public string Email { get; set; }
}
```

`<input asp-for="Email" />` generates:

| Attribute | Value | Source |
|---|---|---|
| `id` | `Email` | Property name |
| `name` | `Email` | Property name (used for model binding) |
| `type` | `email` | `[DataType(DataType.EmailAddress)]` |
| `data-val` | `true` | Any validation attribute present |
| `data-val-required` | `The Customer Email field is required.` | `[Required]` + `[Display]` |
| `data-val-email` | `The Customer Email field is not a valid e-mail address.` | `[EmailAddress]` (implicit from DataType) |

### Nested Properties

```cshtml
@model OrderViewModel

@* For a nested property *@
<input asp-for="ShippingAddress.City" />
@* Generates: name="ShippingAddress.City" id="ShippingAddress_City" *@
```

### Collection Indexing

```cshtml
@for (int i = 0; i < Model.Items.Count; i++)
{
    <input asp-for="Items[i].ProductName" />
    <input asp-for="Items[i].Quantity" />
}
@* Generates: name="Items[0].ProductName", name="Items[0].Quantity", etc. *@
```

This is critical for model binding collections on the server side.

> [!tip] Practical Tip
> The `name` attribute generated by `asp-for` must exactly match the model binding path. If you manually set `name` on an input, model binding may fail. Let `asp-for` handle the name generation unless you have a specific reason to override it.

> [!summary] Section Summary
> - `asp-for` generates `id`, `name`, `type`, and `data-val-*` attributes from the model property
> - Nested properties produce dot-notation names: `ShippingAddress.City`
> - Collection indexing produces bracket-notation: `Items[0].ProductName`
> - Data annotations drive both the HTML type and validation attributes
> - Let `asp-for` generate `name` attributes -- manual overrides break model binding

---

## Custom Tag Helpers

You can create your own tag helpers by inheriting from `TagHelper` and overriding `Process()` or `ProcessAsync()`.

### Basic Structure

```csharp
using Microsoft.AspNetCore.Razor.TagHelpers;

[HtmlTargetElement("alert")]  // Targets <alert> elements
public class AlertTagHelper : TagHelper
{
    public string Type { get; set; } = "info";  // Maps to the "type" attribute

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        // Change the HTML element from <alert> to <div>
        output.TagName = "div";

        // Map our type to Bootstrap classes
        var cssClass = Type.ToLower() switch
        {
            "warning" => "alert alert-warning",
            "danger" or "error" => "alert alert-danger",
            "success" => "alert alert-success",
            _ => "alert alert-info"
        };

        output.Attributes.SetAttribute("class", cssClass);
        output.Attributes.SetAttribute("role", "alert");

        // The child content of <alert>...</alert> is preserved automatically
    }
}
```

Usage:

```cshtml
<alert type="warning">
    <strong>Warning!</strong> Your subscription expires in 3 days.
</alert>
```

Renders:

```html
<div class="alert alert-warning" role="alert">
    <strong>Warning!</strong> Your subscription expires in 3 days.
</div>
```

### Targeting Existing HTML Elements

Instead of creating a custom element, you can target existing HTML elements with specific attributes:

```csharp
[HtmlTargetElement("a", Attributes = "asp-confirm")]
public class ConfirmLinkTagHelper : TagHelper
{
    [HtmlAttributeName("asp-confirm")]
    public string ConfirmMessage { get; set; }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.Attributes.SetAttribute("onclick",
            $"return confirm('{ConfirmMessage}');");
        output.Attributes.RemoveAll("asp-confirm");
    }
}
```

Usage:

```cshtml
<a asp-controller="Products" asp-action="Delete"
   asp-route-id="@product.Id"
   asp-confirm="Are you sure you want to delete this product?">
    Delete
</a>
```

### Async Tag Helpers

For tag helpers that need to read child content:

```csharp
[HtmlTargetElement("markdown")]
public class MarkdownTagHelper : TagHelper
{
    public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
        var childContent = await output.GetChildContentAsync();
        var markdownText = childContent.GetContent();

        // Convert markdown to HTML (using a library like Markdig)
        var html = Markdig.Markdown.ToHtml(markdownText);

        output.TagName = "div";
        output.Attributes.SetAttribute("class", "markdown-content");
        output.Content.SetHtmlContent(html);
    }
}
```

### The [HtmlTargetElement] Attribute

| Property | Purpose | Example |
|---|---|---|
| Tag name (positional) | Which element to target | `[HtmlTargetElement("alert")]` |
| `Attributes` | Only target elements with these attributes | `[HtmlTargetElement("a", Attributes = "asp-confirm")]` |
| `TagStructure` | Self-closing or with content | `TagStructure.NormalOrSelfClosing` |
| `ParentTag` | Only target when inside a specific parent | `[HtmlTargetElement("li", ParentTag = "ul")]` |

> [!warning] Common Misconception
> Tag helper properties are mapped from HTML attributes using **kebab-case to PascalCase** conversion. A C# property `MaxItems` is set via the attribute `max-items`. You can override this with `[HtmlAttributeName("custom-name")]`.

> [!summary] Section Summary
> - Custom tag helpers inherit from `TagHelper` and override `Process()` or `ProcessAsync()`
> - `[HtmlTargetElement]` specifies which element(s) the tag helper targets
> - Tag helper properties automatically map from kebab-case HTML attributes to PascalCase C# properties
> - You can create new elements (`<alert>`) or enhance existing ones (`<a asp-confirm="...">`)
> - `ProcessAsync` is needed when reading child content

---

## Real-World Example: Bootstrap Alert Tag Helper

A more complete custom tag helper with dismissibility, icons, and a heading:

```csharp
using Microsoft.AspNetCore.Razor.TagHelpers;

[HtmlTargetElement("alert")]
public class AlertTagHelper : TagHelper
{
    /// <summary>
    /// Bootstrap alert type: primary, secondary, success, danger, warning, info, light, dark
    /// </summary>
    public string Type { get; set; } = "info";

    /// <summary>
    /// If true, adds a dismiss button to the alert.
    /// </summary>
    public bool Dismissible { get; set; } = false;

    /// <summary>
    /// Optional heading displayed in bold above the message.
    /// </summary>
    public string Heading { get; set; }

    public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "div";

        var cssClass = $"alert alert-{Type}";
        if (Dismissible)
        {
            cssClass += " alert-dismissible fade show";
        }

        output.Attributes.SetAttribute("class", cssClass);
        output.Attributes.SetAttribute("role", "alert");

        // Build inner content
        if (!string.IsNullOrEmpty(Heading))
        {
            output.PreContent.SetHtmlContent($"<h4 class=\"alert-heading\">{Heading}</h4>");
        }

        // Child content is preserved automatically between PreContent and PostContent

        if (Dismissible)
        {
            output.PostContent.SetHtmlContent(
                "<button type=\"button\" class=\"btn-close\" data-bs-dismiss=\"alert\" aria-label=\"Close\"></button>");
        }
    }
}
```

Usage:

```cshtml
@* Simple alert *@
<alert type="success">Product saved successfully.</alert>

@* Dismissible alert with heading *@
<alert type="warning" heading="Subscription Expiring" dismissible="true">
    Your subscription expires on <strong>@Model.ExpirationDate.ToString("MMMM dd, yyyy")</strong>.
    <a href="/account/renew" class="alert-link">Renew now</a> to avoid interruption.
</alert>

@* Error alert *@
<alert type="danger" heading="Error">
    @foreach (var error in Model.Errors)
    {
        <p class="mb-0">@error</p>
    }
</alert>
```

Register in `_ViewImports.cshtml`:

```cshtml
@addTagHelper *, MyApp
```

> [!summary] Section Summary
> - Custom tag helpers can encapsulate complex Bootstrap patterns into clean, semantic HTML
> - `PreContent`, `PostContent`, and `Content` control where generated HTML is placed relative to child content
> - Properties like `Dismissible` and `Heading` provide a clean API surface
> - A single `@addTagHelper *, MyApp` registration enables all custom tag helpers in the assembly

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Tag helpers** are server-side C# classes that enhance HTML elements with `asp-*` attributes, processed on the server and stripped from the final output. They replace the older `@Html.*` methods with a more natural, HTML-like syntax that is readable by both developers and designers.
>
> The **built-in tag helpers** cover the most common scenarios: the **anchor tag helper** (`asp-controller`, `asp-action`, `asp-route-{param}`) generates correct URLs from routing; the **form tag helper** adds `action` URLs and anti-forgery tokens; the **input tag helper** (`asp-for`) generates `name`, `id`, `type`, and `data-val-*` validation attributes from model properties and their data annotations; **label**, **select**, and **textarea** tag helpers bind similarly; and **validation tag helpers** (`asp-validation-for`, `asp-validation-summary`) display error messages.
>
> Utility tag helpers add **cache-busting** (`asp-append-version="true"` on images, CSS, and scripts), **environment-conditional rendering** (`<environment include="Development">`), **partial view rendering** (`<partial name="...">`), and **output caching** (`<cache>`, `<distributed-cache>`).
>
> The `asp-for` attribute is the most powerful mechanism, bridging C# model metadata to HTML attributes. It handles nested properties (`ShippingAddress.City`), collection indexing (`Items[0].Quantity`), and drives both server-side and client-side validation through data annotations.
>
> **Custom tag helpers** inherit from `TagHelper`, target specific elements via `[HtmlTargetElement]`, and override `Process()` or `ProcessAsync()` to modify the rendered output. They enable teams to create semantic, reusable HTML components (like `<alert type="warning">`) that encapsulate complex markup patterns.

---

## Related Topics

- [[Razor Syntax]] -- the `@` syntax foundation that tag helpers build upon
- [[Layouts and Sections]] -- layouts use tag helpers extensively for navigation and asset references
- [[Partial Views and View Components]] -- the `<partial>` and `<vc:>` tag helpers
- [[Razor Pages]] -- Razor Pages use the same tag helpers with `asp-page` and `asp-page-handler`
- [[17.05 - Routing]] -- tag helpers generate URLs based on the routing system
- [[17.06 - Controllers and Actions]] -- `asp-controller` and `asp-action` target controller actions
