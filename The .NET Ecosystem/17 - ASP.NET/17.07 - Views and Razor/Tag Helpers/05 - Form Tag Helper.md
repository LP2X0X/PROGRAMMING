---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


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
