---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


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
