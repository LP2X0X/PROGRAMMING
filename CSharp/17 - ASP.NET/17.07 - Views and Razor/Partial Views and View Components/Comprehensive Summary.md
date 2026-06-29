---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


> [!tip] Complete Summary
> ASP.NET Core offers two complementary mechanisms for reusable view fragments: **partial views** and **view components**.
>
> **Partial views** are simple `.cshtml` files that render HTML from data provided by the parent view. They are invoked with `<partial name="_Name" model="data" />`, searched in the current folder then `/Views/Shared/`, and follow the `_Prefix` naming convention. They are ideal for repeated UI patterns like product cards, form fields, and list items where all data is already available. They should contain only presentation logic.
>
> **View components** are self-contained units with a C# class (`ViewComponent` subclass) and a Razor view (`/Views/Shared/Components/[Name]/Default.cshtml`). The class implements `InvokeAsync()` to fetch data via injected services, then returns `View(model)`. They are invoked with the `<vc:name />` tag helper or `Component.InvokeAsync()`. View components are the right choice when a UI fragment needs its own data pipeline -- navigation menus, cart widgets, recommendation engines, and any widget that cannot rely on the parent view for its data.
>
> The decision is simple: if the fragment **displays** pre-prepared data, use a partial view. If the fragment **fetches** its own data, use a view component. View components additionally offer full DI support, testability through constructor injection, and the ability to implement caching and complex logic within the `InvokeAsync()` method.

---
