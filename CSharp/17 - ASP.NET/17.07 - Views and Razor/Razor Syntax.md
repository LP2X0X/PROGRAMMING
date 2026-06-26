---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---

# Razor Syntax

> [!ad-note] Overview
> Razor is a markup syntax that lets you embed C# code into HTML using the `@` character as a transition marker. It powers views in ASP.NET Core MVC and [[Razor Pages]], providing a clean, concise way to generate dynamic HTML on the server. This note covers every aspect of Razor syntax from basic expressions through directives and special files.

## Table of Contents

- [What Razor Is](#what-razor-is)
- [Implicit Expressions](#implicit-expressions)
- [Explicit Expressions](#explicit-expressions)
- [Code Blocks](#code-blocks)
- [Control Flow Statements](#control-flow-statements)
- [Literal Text Output in Code Blocks](#literal-text-output-in-code-blocks)
- [HTML Encoding and XSS Prevention](#html-encoding-and-xss-prevention)
- [Razor Comments](#razor-comments)
- [Razor Directives](#razor-directives)
- [Functions and Code Blocks in Views](#functions-and-code-blocks-in-views)
- [Shared View Files](#shared-view-files)
- [Escaping the @ Character](#escaping-the--character)
- [Real-World Example: Product Detail Page](#real-world-example-product-detail-page)
- [Comprehensive Summary](#comprehensive-summary)
- [Related Topics](#related-topics)

---

## What Razor Is

**Razor** is a server-side markup syntax developed by Microsoft that allows you to embed C# code directly into HTML files. The Razor engine parses `.cshtml` files (or `.razor` for Blazor components) and compiles them into C# classes that produce HTML output at runtime.

The fundamental concept is the **`@` transition character**. The Razor parser uses `@` to switch from HTML mode to C# mode. The parser is smart enough to figure out where the C# expression ends and HTML begins again, which is what makes Razor feel natural compared to older syntaxes like Web Forms (`<%= %>`).

Razor files in ASP.NET Core MVC live under the `/Views/` folder, organized by controller:

```
/Views/
    /Home/
        Index.cshtml
        About.cshtml
    /Products/
        Index.cshtml
        Details.cshtml
    /Shared/
        _Layout.cshtml
        _ValidationScriptsPartial.cshtml
    _ViewImports.cshtml
    _ViewStart.cshtml
```

> [!ad-note] Razor vs Blazor Components
> Razor syntax is used in two distinct contexts: **Razor Views** (`.cshtml`, server-rendered MVC/Razor Pages) and **Razor Components** (`.razor`, Blazor). While they share the same core syntax, Blazor components add event handling, component parameters, and an interactive rendering model. This note focuses on Razor in the MVC/Razor Pages context.

> [!summary] Section Summary
> - Razor is a server-side markup syntax using `@` to transition between HTML and C#
> - Files use the `.cshtml` extension and compile to C# classes at runtime
> - Views are organized under `/Views/` by controller name
> - Razor is distinct from Blazor's `.razor` components, though they share syntax foundations

---

## Implicit Expressions

**Implicit expressions** are the simplest form of Razor syntax. You prefix a C# expression with `@` and Razor automatically determines where the expression ends by following C# identifier rules (letters, digits, dots, brackets).

```cshtml
<h1>@Model.ProductName</h1>
<p>Current time: @DateTime.Now</p>
<p>Total items: @Model.Items.Count</p>
<p>Category: @Model.Category.Name</p>
```

The Razor parser follows the **dot chain** until it encounters something that is not a valid C# continuation (like a space, HTML tag, or certain punctuation).

> [!warning] Common Misconception
> Many developers assume `@Model.Price + tax` will compute a sum. It will not. Razor sees `@Model.Price` as the expression and renders the rest as literal text. The output would be something like `19.99 + tax`. You need [[#Explicit Expressions]] for compound expressions.

**What works with implicit expressions:**
- Property access: `@Model.Name`
- Method calls: `@DateTime.Now.ToString("yyyy-MM-dd")`
- Indexer access: `@Model.Items[0]`
- Chained members: `@Model.Category.Products.Count`

**What does NOT work:**
- Arithmetic: `@Model.Price * 1.1` -- Razor stops at the space before `*`
- String concatenation with `+`
- Ternary operators
- Anything requiring parentheses around the whole expression

> [!ad-note] C# Keywords as Identifiers
> Implicit expressions cannot start with C# keywords. `@class` would confuse the parser because `class` is a keyword. Use explicit expressions for these: `@(myClass)`.

> [!summary] Section Summary
> - Implicit expressions start with `@` followed by a C# expression
> - The parser follows valid C# identifiers (dots, brackets, method calls)
> - They work for simple property access, method calls, and indexers
> - They do NOT work for arithmetic, ternary operators, or compound expressions
> - Use explicit expressions when Razor cannot determine the expression boundary

---

## Explicit Expressions

**Explicit expressions** use `@(...)` to clearly delineate where the C# expression starts and ends. Use them whenever the expression is more complex than a simple property chain.

```cshtml
@* Arithmetic *@
<p>Price with tax: @(Model.Price * 1.1)</p>

@* String concatenation *@
<p>Full name: @(Model.FirstName + " " + Model.LastName)</p>

@* Ternary operator *@
<p class="@(Model.IsActive ? "active" : "inactive")">Status</p>

@* Disambiguation from email addresses *@
<p>Contact: support@(company).com</p>

@* Generic types *@
<p>@(new List<string> { "A", "B", "C" }.Count)</p>
```

> [!tip] Practical Tip
> One of the most common uses for explicit expressions is embedding C# in HTML attributes. Without parentheses, Razor might misinterpret where the expression ends:
> ```cshtml
> @* Wrong -- Razor sees @Model.CssClass as the whole attribute value *@
> <div class="container @Model.CssClass-modifier">
>
> @* Correct -- explicit boundary *@
> <div class="container @(Model.CssClass)-modifier">
> ```

**The email problem** is a classic gotcha. Razor is smart enough to recognize email patterns and will NOT try to parse `user@example.com` as a Razor expression. But if you genuinely want to insert a C# expression that looks like an email, use explicit syntax: `user@(domainVariable).com`.

> [!summary] Section Summary
> - Explicit expressions use `@(...)` to wrap complex C# expressions
> - Required for arithmetic, ternary operators, string concatenation, and generics
> - Essential for disambiguating expressions inside HTML attributes
> - Razor auto-detects email patterns, but explicit syntax overrides this when needed

---

## Code Blocks

**Code blocks** let you write multi-line C# logic inside a view using `@{ ... }`. Code inside a code block executes but does not produce output unless you explicitly write to the response or switch back to HTML mode.

```cshtml
@{
    var greeting = "Welcome";
    var currentHour = DateTime.Now.Hour;
    string timeOfDay;

    if (currentHour < 12)
    {
        timeOfDay = "morning";
    }
    else if (currentHour < 17)
    {
        timeOfDay = "afternoon";
    }
    else
    {
        timeOfDay = "evening";
    }
}

<h1>@greeting, good @timeOfDay!</h1>
```

Variables declared inside a code block are available for the rest of the view (they compile into the same method). You can have multiple code blocks in a single view, and variables from earlier blocks are accessible in later ones.

> [!warning] Common Misconception
> Code blocks should contain **view logic**, not business logic. If you find yourself writing database queries, complex calculations, or business rules in a code block, that logic belongs in the controller, service layer, or [[Partial Views and View Components|view component]]. Views should focus on presentation.

**Mixing HTML and C# inside code blocks:**

```cshtml
@{
    var products = Model.Products;

    if (!products.Any())
    {
        <p class="empty-state">No products found.</p>
    }
}
```

When you are inside a code block (`@{ }`), Razor switches to C# mode. It recognizes HTML tags and automatically transitions back to HTML mode when it encounters a `<tag>`. This is how you can freely mix HTML into C# logic.

> [!summary] Section Summary
> - Code blocks use `@{ ... }` for multi-line C# statements
> - Variables declared in code blocks are available throughout the rest of the view
> - HTML tags inside code blocks automatically transition Razor back to HTML mode
> - Keep code blocks focused on presentation logic, not business logic

---

## Control Flow Statements

Razor provides shorthand for common C# control flow structures. These are prefixed with `@` but do NOT need curly braces around the keyword -- Razor recognizes the C# keywords.

### @if / @else if / @else

```cshtml
@if (Model.Products.Any())
{
    <ul>
        @foreach (var product in Model.Products)
        {
            <li>@product.Name - @product.Price.ToString("C")</li>
        }
    </ul>
}
else if (Model.IsLoading)
{
    <p>Loading products...</p>
}
else
{
    <p>No products available.</p>
}
```

> [!ad-note] Key Detail
> Notice that `else if` and `else` do NOT need a leading `@`. They are continuations of the `@if` block. Adding `@else` works too, but it is not required.

### @switch

```cshtml
@switch (Model.Status)
{
    case OrderStatus.Pending:
        <span class="badge bg-warning">Pending</span>
        break;
    case OrderStatus.Shipped:
        <span class="badge bg-info">Shipped</span>
        break;
    case OrderStatus.Delivered:
        <span class="badge bg-success">Delivered</span>
        break;
    default:
        <span class="badge bg-secondary">Unknown</span>
        break;
}
```

### @for

```cshtml
@for (int i = 0; i < Model.TopProducts.Count; i++)
{
    <div class="product-rank">
        <span class="rank">#@(i + 1)</span>
        <span class="name">@Model.TopProducts[i].Name</span>
    </div>
}
```

### @foreach

```cshtml
@foreach (var category in Model.Categories)
{
    <div class="category-card">
        <h3>@category.Name</h3>
        <p>@category.Description</p>
        <span>@category.ProductCount products</span>
    </div>
}
```

### @while and @do...while

```cshtml
@{
    var index = 0;
}
@while (index < 5)
{
    <p>Item @index</p>
    index++;
}
```

### @try / catch / finally

```cshtml
@try
{
    <p>@Model.RiskyProperty</p>
}
catch (Exception ex)
{
    <p class="text-danger">Error: @ex.Message</p>
}
```

> [!warning] Common Misconception
> Using `try/catch` in views is almost always a code smell. If a property might be null or throw, handle that in the controller or model. Views should receive clean, ready-to-display data.

### @using (Disposable Scope)

Not to be confused with the `@using` namespace directive, the `@using` statement creates a scope that disposes an `IDisposable` object:

```cshtml
@using (Html.BeginForm("Search", "Products", FormMethod.Get))
{
    <input type="text" name="query" />
    <button type="submit">Search</button>
}
```

This is less common now that [[Tag Helpers]] provide a cleaner syntax for forms.

> [!summary] Section Summary
> - `@if`, `@switch`, `@for`, `@foreach`, `@while` work as expected with HTML inside their bodies
> - `else` and `else if` do not require a leading `@`
> - `@try/catch` works but is a code smell in views
> - `@using` creates disposable scopes (distinct from the namespace directive)
> - All control flow blocks allow free mixing of HTML and C# inside their bodies

---

## Literal Text Output in Code Blocks

When you are inside a code block and want to output plain text (not wrapped in an HTML tag), Razor needs a hint that you are switching to content mode. There are two mechanisms: `@:` and `<text>`.

### The @: Operator (Single Line)

`@:` tells Razor "treat the rest of this line as literal content output":

```cshtml
@if (Model.ShowGreeting)
{
    @:Hello, this is plain text output without any HTML tag.
}
```

### The `<text>` Element (Multi-Line)

`<text>` is a pseudo-element recognized by Razor. It is **not rendered** to the browser -- it simply tells Razor to treat its contents as text output:

```cshtml
@for (int i = 0; i < 3; i++)
{
    <text>
        Item number @i
        is being processed.
    </text>
}
```

> [!tip] Practical Tip
> You rarely need `@:` or `<text>` in practice. If your content is wrapped in any HTML tag (even a `<span>`), Razor transitions automatically. These are only needed for outputting raw text inside a code block without any wrapping element.

> [!summary] Section Summary
> - `@:` outputs the rest of the current line as literal content (single line)
> - `<text>` wraps multiple lines of literal content output (multi-line)
> - Both are only needed when outputting text inside a code block without an HTML wrapper
> - If your content is inside any HTML element, Razor transitions automatically

---

## HTML Encoding and XSS Prevention

One of Razor's most important security features is **automatic HTML encoding**. Every Razor expression (`@Model.Something`) encodes the output so that special characters like `<`, `>`, `&`, and `"` are converted to their HTML entity equivalents.

```cshtml
@{
    var userInput = "<script>alert('XSS')</script>";
}

@* This is SAFE -- outputs encoded text: *@
<p>@userInput</p>
@* Renders as: &lt;script&gt;alert('XSS')&lt;/script&gt; *@

@* This is DANGEROUS -- outputs raw unencoded HTML: *@
<p>@Html.Raw(userInput)</p>
@* Renders as: <script>alert('XSS')</script> *@
```

> [!danger] Security Warning
> `@Html.Raw()` bypasses HTML encoding entirely. Only use it when you are **absolutely certain** the content is safe -- for example, HTML generated by your own server-side code (like a Markdown-to-HTML converter) or content from a trusted CMS that has been sanitized. **Never** use `Html.Raw()` on user-provided input.

### When to Use Html.Raw()

Legitimate uses of `Html.Raw()`:
- Rendering HTML from a rich-text editor that has been sanitized server-side
- Outputting pre-built HTML from a trusted source
- Inserting JSON data into a `<script>` tag (though even this should use `Json.Serialize()`)

```cshtml
@* Safe: server-generated HTML from a sanitized Markdown renderer *@
<div class="article-content">
    @Html.Raw(Model.SanitizedHtmlContent)
</div>

@* Safe: serialized JSON for JavaScript consumption *@
<script>
    var config = @Json.Serialize(Model.ClientConfig);
</script>
```

> [!summary] Section Summary
> - Razor automatically HTML-encodes all output to prevent XSS attacks
> - `@Html.Raw()` bypasses encoding -- use with extreme caution
> - Only use `Html.Raw()` with trusted, sanitized content
> - `@Json.Serialize()` is the preferred way to pass data to JavaScript

---

## Razor Comments

Razor has its own comment syntax that is stripped from the output entirely (unlike HTML comments which are visible in the page source):

```cshtml
@* This is a Razor comment.
   It can span multiple lines.
   It is NOT sent to the browser. *@

<!-- This is an HTML comment. It IS visible in page source. -->
```

> [!tip] Practical Tip
> Use Razor comments (`@* *@`) for development notes and TODOs. Use HTML comments (`<!-- -->`) only when you intentionally want the comment visible in the rendered HTML source (rare in production).

You can also use C# comments inside code blocks:

```cshtml
@{
    // Single-line C# comment
    var name = Model.Name;

    /* Multi-line
       C# comment */
    var price = Model.Price;
}
```

> [!summary] Section Summary
> - Razor comments use `@* ... *@` and are completely stripped from the output
> - HTML comments `<!-- -->` are sent to the browser and visible in page source
> - Standard C# comments (`//` and `/* */`) work inside code blocks
> - Prefer Razor comments over HTML comments for development notes

---

## Razor Directives

**Directives** are special Razor instructions that begin with `@` followed by a keyword. They change how the view is parsed or compiled.

### @model -- Declaring the View's Model Type

The most important directive. It tells the view what type `Model` is, enabling IntelliSense and compile-time type checking:

```cshtml
@model ProductDetailViewModel

<h1>@Model.Name</h1>
<p>@Model.Description</p>
<p>Price: @Model.Price.ToString("C")</p>
```

> [!warning] Common Misconception
> `@model` (lowercase) declares the type. `@Model` (uppercase) accesses the instance. Confusing the two is a common error for beginners. `@model` appears once at the top of the file; `@Model` is used throughout to read properties.

### @using -- Importing Namespaces

```cshtml
@using MyApp.Models
@using MyApp.ViewModels
@using System.Globalization

@model ProductDetailViewModel
```

Namespaces added with `@using` in a view apply only to that view. For shared imports, use `_ViewImports.cshtml` (see [[#Shared View Files]]).

### @inject -- Dependency Injection into Views

`@inject` allows you to inject a registered service directly into the view:

```cshtml
@inject IConfiguration Configuration
@inject IStringLocalizer<SharedResource> Localizer

<p>App Version: @Configuration["AppVersion"]</p>
<p>@Localizer["WelcomeMessage"]</p>
```

> [!tip] Practical Tip
> Use `@inject` sparingly. If a view needs complex data that requires service calls, consider using a [[Partial Views and View Components|View Component]] instead. `@inject` is appropriate for simple cross-cutting concerns like localization or configuration values.

### @addTagHelper and @removeTagHelper

These enable or disable [[Tag Helpers]]:

```cshtml
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper MyApp.TagHelpers.*, MyApp
@removeTagHelper Microsoft.AspNetCore.Mvc.TagHelpers.EnvironmentTagHelper, Microsoft.AspNetCore.Mvc.TagHelpers
```

Typically placed in `_ViewImports.cshtml` so all views benefit.

### @tagHelperPrefix

Requires all tag helpers to use a prefix, reducing confusion between tag helpers and plain HTML:

```cshtml
@tagHelperPrefix th:
```

After this directive, you would write `<th:a asp-controller="Home">` instead of `<a asp-controller="Home">`.

### @attribute

Applies a C# attribute to the generated class:

```cshtml
@attribute [Authorize]
```

### @namespace

Overrides the namespace of the generated class (rarely used).

> [!summary] Section Summary
> - `@model` declares the strongly-typed model (lowercase `m` = type, uppercase `M` = instance)
> - `@using` imports namespaces for a single view
> - `@inject` provides dependency injection directly into views (use sparingly)
> - `@addTagHelper` / `@removeTagHelper` control tag helper availability
> - Directives typically go at the top of the file, before any HTML

---

## Functions and Code Blocks in Views

The `@functions` directive (or `@code` in Blazor) lets you define methods and properties inside the view:

```cshtml
@functions {
    public string GetCssClass(decimal price)
    {
        if (price > 100) return "text-danger fw-bold";
        if (price > 50) return "text-warning";
        return "text-success";
    }

    public string FormatDiscount(decimal original, decimal discounted)
    {
        var percentage = (1 - discounted / original) * 100;
        return $"{percentage:F0}% off";
    }
}

@foreach (var product in Model.Products)
{
    <div class="product">
        <span class="@GetCssClass(product.Price)">@product.Price.ToString("C")</span>
        @if (product.DiscountedPrice.HasValue)
        {
            <span class="discount">
                @FormatDiscount(product.Price, product.DiscountedPrice.Value)
            </span>
        }
    </div>
}
```

> [!warning] Common Misconception
> Just because you *can* define methods in views does not mean you *should*. `@functions` is appropriate for small formatting helpers specific to one view. If the logic is reusable or complex, it belongs in a view model, extension method, or [[Partial Views and View Components|view component]]. Overuse of `@functions` leads to "fat views" that are hard to test and maintain.

> [!summary] Section Summary
> - `@functions { }` defines C# methods and properties inside a view
> - Appropriate for small, view-specific formatting helpers
> - Complex or reusable logic should live in view models or services
> - `@code { }` is the Blazor equivalent (not used in MVC views)

---

## Shared View Files

Two special files control shared behavior across all views in a folder (or the entire application).

### _ViewImports.cshtml

`_ViewImports.cshtml` contains directives that are automatically applied to every view in the same directory and all subdirectories:

```cshtml
@* /Views/_ViewImports.cshtml *@
@using MyApp
@using MyApp.Models
@using MyApp.ViewModels
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper MyApp.TagHelpers.*, MyApp
```

**Key behaviors:**
- Directives cascade downward through subdirectories
- You can place `_ViewImports.cshtml` at multiple levels -- they are additive
- Supported directives: `@using`, `@addTagHelper`, `@removeTagHelper`, `@tagHelperPrefix`, `@model`, `@inject`

A common pattern is a root-level `_ViewImports.cshtml` for application-wide imports, and folder-level ones for area-specific imports.

### _ViewStart.cshtml

`_ViewStart.cshtml` contains C# code that runs before every view renders. Its primary use is setting the default [[Layouts and Sections|layout]]:

```cshtml
@* /Views/_ViewStart.cshtml *@
@{
    Layout = "_Layout";
}
```

**Key behaviors:**
- Runs before the view's own code
- Can contain conditional logic (e.g., different layouts per area)
- Cascades downward like `_ViewImports.cshtml`
- Can be placed at multiple directory levels

```cshtml
@* /Views/Admin/_ViewStart.cshtml *@
@{
    Layout = "_AdminLayout";
}
```

> [!ad-note] Execution Order
> When a view renders: `_ViewStart.cshtml` runs first (setting layout and other defaults), then the view itself executes, then the layout wraps the result. If there are nested `_ViewStart.cshtml` files, they run from outermost to innermost.

> [!summary] Section Summary
> - `_ViewImports.cshtml` provides shared `@using`, `@addTagHelper`, and other directives to all views in its directory tree
> - `_ViewStart.cshtml` runs code before every view (typically sets the default layout)
> - Both cascade downward through subdirectories and can exist at multiple levels
> - The underscore prefix is a convention indicating these are not directly renderable views

---

## Escaping the @ Character

Since `@` is the Razor transition character, you need to escape it when you want a literal `@` in the output:

```cshtml
@* Double @@ produces a single literal @ *@
<p>Email: user@@example.com</p>

@* Output: Email: user@example.com *@
```

> [!ad-note] Smart Parsing
> Razor is intelligent about email addresses. If you write `user@example.com` in HTML context, Razor recognizes it as an email (not a C# expression) and outputs it literally. You typically only need `@@` when the context is ambiguous -- for example, inside attribute values or when the text after `@` looks like a valid C# identifier.

Use `@@` in these situations:
- Inside CSS `@@media` queries written inline (rare, but happens)
- Twitter/social handles: `@@username`
- Any context where Razor might misinterpret `@` followed by an identifier

> [!summary] Section Summary
> - `@@` produces a literal `@` in the output
> - Razor auto-detects email addresses and does not try to parse them
> - Use `@@` when the context is ambiguous or when `@` is followed by a valid C# identifier

---

## Real-World Example: Product Detail Page

Putting it all together -- a complete product detail view using the syntax covered above:

```cshtml
@model MyApp.ViewModels.ProductDetailViewModel
@inject IStringLocalizer<ProductResource> Localizer

@{
    ViewData["Title"] = Model.Product.Name;
    var hasDiscount = Model.Product.DiscountedPrice.HasValue;
    var savingsPercent = hasDiscount
        ? (1 - Model.Product.DiscountedPrice.Value / Model.Product.Price) * 100
        : 0;
}

<nav aria-label="breadcrumb">
    <ol class="breadcrumb">
        <li class="breadcrumb-item">
            <a asp-controller="Home" asp-action="Index">Home</a>
        </li>
        <li class="breadcrumb-item">
            <a asp-controller="Products" asp-action="Index">Products</a>
        </li>
        <li class="breadcrumb-item">
            <a asp-controller="Products" asp-action="ByCategory"
               asp-route-id="@Model.Product.CategoryId">
                @Model.Product.CategoryName
            </a>
        </li>
        <li class="breadcrumb-item active">@Model.Product.Name</li>
    </ol>
</nav>

<div class="row">
    <div class="col-md-6">
        <img src="@Model.Product.ImageUrl"
             alt="@Model.Product.Name"
             class="img-fluid rounded" />
    </div>
    <div class="col-md-6">
        <h1>@Model.Product.Name</h1>
        <p class="text-muted">@Model.Product.Brand</p>

        @if (hasDiscount)
        {
            <p>
                <span class="text-decoration-line-through text-muted">
                    @Model.Product.Price.ToString("C")
                </span>
                <span class="text-danger fs-4 fw-bold ms-2">
                    @Model.Product.DiscountedPrice.Value.ToString("C")
                </span>
                <span class="badge bg-danger ms-2">
                    Save @(savingsPercent.ToString("F0"))%
                </span>
            </p>
        }
        else
        {
            <p class="fs-4 fw-bold">@Model.Product.Price.ToString("C")</p>
        }

        <div class="product-description">
            @Html.Raw(Model.Product.SanitizedDescriptionHtml)
        </div>

        @if (Model.Product.StockCount > 0)
        {
            <p class="text-success">
                @Localizer["InStock"] (@Model.Product.StockCount @Localizer["Available"])
            </p>
            <form asp-controller="Cart" asp-action="Add" method="post">
                <input type="hidden" asp-for="Product.Id" />
                <div class="input-group mb-3" style="max-width: 200px;">
                    <input type="number" name="quantity" value="1"
                           min="1" max="@Model.Product.StockCount"
                           class="form-control" />
                    <button type="submit" class="btn btn-primary">
                        @Localizer["AddToCart"]
                    </button>
                </div>
            </form>
        }
        else
        {
            <p class="text-danger">@Localizer["OutOfStock"]</p>
        }
    </div>
</div>

@* Related products section *@
@if (Model.RelatedProducts.Any())
{
    <hr />
    <h2>@Localizer["RelatedProducts"]</h2>
    <div class="row">
        @foreach (var related in Model.RelatedProducts)
        {
            <div class="col-md-3 mb-3">
                <partial name="_ProductCard" model="related" />
            </div>
        }
    </div>
}

@section Scripts {
    <script>
        var productConfig = @Json.Serialize(new {
            productId = Model.Product.Id,
            maxQuantity = Model.Product.StockCount
        });
    </script>
}
```

This example demonstrates:
- `@model` directive with a strongly-typed view model
- `@inject` for localization
- Code blocks for computed values
- `@if`/`@else` for conditional rendering
- `@foreach` for iteration
- Implicit and explicit expressions
- `@Html.Raw()` for pre-sanitized HTML
- `@Json.Serialize()` for passing data to JavaScript
- [[Tag Helpers]] (`asp-controller`, `asp-action`, `asp-for`, `asp-route-id`)
- [[Partial Views and View Components|Partial views]] for reusable card components
- [[Layouts and Sections|Sections]] (`@section Scripts`) for script placement

> [!summary] Section Summary
> - A real-world Razor view combines multiple syntax features into a cohesive page
> - Strong typing with `@model` provides IntelliSense and compile-time checking
> - Conditional rendering, iteration, and computed values live naturally alongside HTML
> - Tag helpers, partial views, and sections integrate seamlessly with Razor syntax

---

## Comprehensive Summary

> [!tip] Complete Summary
> **Razor** is ASP.NET Core's server-side markup syntax that uses `@` to embed C# into HTML. It comes in two expression forms: **implicit** (`@Model.Name` for simple property chains) and **explicit** (`@(expression)` for complex calculations, ternary operators, and disambiguation). **Code blocks** (`@{ }`) allow multi-line C# logic, while **control flow** keywords (`@if`, `@foreach`, `@switch`, `@for`) let you conditionally render HTML. When outputting plain text inside code blocks, use `@:` for single lines or `<text>` for multiple lines.
>
> Razor's **automatic HTML encoding** is a critical security feature that prevents XSS attacks -- all output is encoded by default, and `@Html.Raw()` should only be used with sanitized, trusted content. **Comments** (`@* *@`) are stripped entirely from output, unlike HTML comments.
>
> **Directives** configure the view: `@model` declares the strongly-typed model (enabling IntelliSense), `@using` imports namespaces, `@inject` provides DI access, and `@addTagHelper` enables tag helpers. The `@functions` directive allows defining helper methods in the view, though this should be used sparingly.
>
> Two special shared files govern behavior across views: **`_ViewImports.cshtml`** shares directives (imports, tag helpers) across all views in its directory tree, while **`_ViewStart.cshtml`** runs code before each view (typically setting the default layout). Both cascade downward through subdirectories. Finally, `@@` produces a literal `@` character, though Razor is smart enough to auto-detect email patterns.

---

## Related Topics

- [[Layouts and Sections]] -- master page structure with `@RenderBody()` and `@RenderSection()`
- [[Partial Views and View Components]] -- reusable view fragments and mini-controllers
- [[Tag Helpers]] -- server-side HTML element processing
- [[Razor Pages]] -- page-focused alternative to MVC controllers
- [[17.06 - Controllers and Actions]] -- the C in MVC that prepares data for views
- [[17.03 - Dependency Injection]] -- the service registration that `@inject` draws from
