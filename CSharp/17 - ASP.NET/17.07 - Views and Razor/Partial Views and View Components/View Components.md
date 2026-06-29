---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


A **view component** is a self-contained unit that combines:
1. A **C# class** (the "mini-controller") that fetches data and prepares a model
2. A **Razor view** that renders the HTML

View components are similar to partial views, but they can independently query databases, call services, and perform logic -- all without the parent view or controller knowing about it.

### When to Use View Components

- Navigation menus that need to load menu items from a database
- Shopping cart summaries that query the cart service
- "Recently viewed" or "recommended products" widgets
- Sidebar content that varies by user role
- Any reusable UI widget that needs its own data pipeline

> [!ad-note] View Components vs Partial Views
> The mental model: if the fragment only *displays* data (already available from the parent), use a partial view. If the fragment needs to *fetch* its own data, use a view component.

> [!summary] Section Summary
> - View components are mini-controllers that produce HTML fragments with their own data
> - They combine a C# class (data fetching) and a Razor view (rendering)
> - Use them when the UI fragment needs independent data access
> - They encapsulate both logic and presentation, unlike partials which are presentation-only

---
