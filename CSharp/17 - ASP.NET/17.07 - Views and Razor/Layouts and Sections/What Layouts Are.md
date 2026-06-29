---
tags:
  - csharp
  - asp-net-core
  - razor
  - layouts
  - views
---


A **layout** is a Razor view that defines the HTML shell shared by multiple pages. Instead of duplicating the `<!DOCTYPE html>`, `<head>`, navigation bar, and footer in every view, you define them once in a layout. Each page view provides only its unique content, which the layout inserts at a designated point.

This is conceptually similar to:
- Template inheritance in Django/Jinja2
- Master pages in ASP.NET Web Forms
- Slots in Vue.js or Svelte

The benefits are straightforward:
- **DRY principle**: common markup lives in one place
- **Consistency**: every page shares the same navigation, footer, and asset references
- **Maintainability**: updating the nav or adding a new CSS file is a single change

> [!summary] Section Summary
> - Layouts define the shared HTML structure (head, nav, footer) for an application
> - Child views provide only their unique content
> - This eliminates duplication and ensures visual consistency across pages

---
