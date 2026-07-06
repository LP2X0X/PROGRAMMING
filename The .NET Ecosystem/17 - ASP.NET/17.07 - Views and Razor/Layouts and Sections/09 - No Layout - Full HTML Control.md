---
tags:
  - csharp
  - asp-net-core
  - razor
  - layouts
  - views
---


Some views need to render a complete HTML document without any layout wrapping. Set `Layout = null`:

```cshtml
@{
    Layout = null;
}

<!DOCTYPE html>
<html>
<head>
    <title>Error</title>
    <style>
        body { font-family: sans-serif; text-align: center; padding: 50px; }
    </style>
</head>
<body>
    <h1>Something went wrong</h1>
    <p>Please try again later.</p>
</body>
</html>
```

Common scenarios for `Layout = null`:
- **Error pages**: the layout itself might be broken, so the error page must be self-contained
- **Login/auth pages**: when they need a completely different HTML structure
- **Email templates**: rendered to HTML strings, not served as web pages
- **Embedded widgets**: HTML fragments served inside iframes
- **PDF generation**: HTML templates that a PDF renderer processes

> [!summary] Section Summary
> - `Layout = null` renders the view as a standalone HTML document with no layout wrapper
> - Appropriate for error pages, email templates, embedded widgets, and PDF generation
> - The view must include the complete HTML structure (`<!DOCTYPE>`, `<html>`, `<head>`, `<body>`)

---
