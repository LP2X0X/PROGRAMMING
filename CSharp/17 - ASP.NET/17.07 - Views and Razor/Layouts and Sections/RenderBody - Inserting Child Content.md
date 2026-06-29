---
tags:
  - csharp
  - asp-net-core
  - razor
  - layouts
  - views
---


`@RenderBody()` is the single most important method in a layout. It marks where the child view's content is inserted. **Every layout must call `@RenderBody()` exactly once.**

```cshtml
@* In _Layout.cshtml *@
<main class="container">
    @RenderBody()
</main>
```

When a child view like `/Views/Products/Details.cshtml` renders:

```cshtml
@model ProductViewModel

<h1>@Model.Name</h1>
<p>@Model.Description</p>
<p>Price: @Model.Price.ToString("C")</p>
```

The final HTML output combines the layout's structure with the child view's content:

```html
<!DOCTYPE html>
<html>
<head>...</head>
<body>
    <header>...</header>
    <main class="container">
        <h1>Widget Pro</h1>
        <p>The finest widget money can buy.</p>
        <p>Price: $29.99</p>
    </main>
    <footer>...</footer>
</body>
</html>
```

> [!warning] Common Misconception
> `@RenderBody()` does not accept parameters. You cannot pass data to it or specify which view to render. It always renders the content of the current child view. If you need multiple replaceable content areas, use **sections**.

> [!summary] Section Summary
> - `@RenderBody()` inserts the child view's entire content at that point in the layout
> - It must be called exactly once per layout
> - It takes no parameters and always renders the current child view
> - For multiple insertion points, use sections

---
