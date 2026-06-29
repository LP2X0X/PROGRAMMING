---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


Partial views hit their limits when the HTML fragment needs its own data. Consider this scenario:

You want to display a "cart summary" widget in the navigation bar (defined in the [[Layouts and Sections|layout]]). The layout does not have cart data -- it does not even know about carts. You would need to:

1. Add cart data to every controller action's view model, OR
2. Use `ViewData`/`ViewBag` and set it in every controller, OR
3. Use an action filter to inject cart data globally

All of these options are clunky and violate separation of concerns. This is exactly the problem **view components** solve.

> [!summary] Section Summary
> - Partial views require the calling view to provide all data
> - When a UI fragment needs its own independent data fetching, partials become impractical
> - Injecting data via ViewBag/ViewData or action filters is a code smell for this scenario
> - View components solve this by encapsulating data access and rendering together

---
