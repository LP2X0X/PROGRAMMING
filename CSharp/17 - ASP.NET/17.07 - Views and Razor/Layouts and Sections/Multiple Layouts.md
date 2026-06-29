---
tags:
  - csharp
  - asp-net-core
  - razor
  - layouts
  - views
---


Real applications often need different layouts for different areas. Common examples:
- Public site layout (marketing header, footer)
- Admin dashboard layout (sidebar navigation, minimal footer)
- Authentication layout (centered card, no navigation)
- Email/print layout (no interactive elements)

```
/Views/Shared/
    _Layout.cshtml           ← public site
    _AdminLayout.cshtml      ← admin dashboard
    _AuthLayout.cshtml       ← login/register pages
    _PrintLayout.cshtml      ← print-friendly pages
```

### Switching Layouts per View

```cshtml
@* /Views/Account/Login.cshtml *@
@{
    Layout = "_AuthLayout";
}

<div class="login-card">
    <h1>Sign In</h1>
    @* login form *@
</div>
```

### Switching Layouts per Area

```cshtml
@* /Areas/Admin/Views/_ViewStart.cshtml *@
@{
    Layout = "_AdminLayout";
}
```

### Switching Layouts per Controller

```cshtml
@* /Views/_ViewStart.cshtml *@
@{
    var controller = Context.Request.RouteValues["controller"]?.ToString();
    Layout = controller == "Account" ? "_AuthLayout" : "_Layout";
}
```

> [!tip] Practical Tip
> Keep layout switching simple. Prefer separate `_ViewStart.cshtml` files in area/folder hierarchies over complex conditional logic. If your `_ViewStart.cshtml` has more than a simple `if/else`, consider reorganizing your view folder structure.

> [!summary] Section Summary
> - Applications commonly have multiple layouts for different areas (public, admin, auth)
> - Layouts can be switched per-view, per-area (via folder-level `_ViewStart.cshtml`), or conditionally
> - Prefer folder-level `_ViewStart.cshtml` files over complex conditionals for cleaner organization

---
