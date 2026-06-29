---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


A **partial view** is a `.cshtml` file that renders a fragment of HTML. It is not a standalone page -- it is meant to be embedded within another view or [[Layouts and Sections|layout]]. Think of partials as reusable HTML components that receive their data from the parent view.

Common uses:
- Product cards in a product listing
- Navigation menus
- Form fragments shared between Create and Edit views
- Comment blocks in a blog
- Table rows with complex formatting

```cshtml
@* /Views/Shared/_ProductCard.cshtml *@
@model Product

<div class="card h-100">
    <img src="@Model.ImageUrl" class="card-img-top" alt="@Model.Name" />
    <div class="card-body">
        <h5 class="card-title">@Model.Name</h5>
        <p class="card-text text-muted">@Model.Category.Name</p>
        <p class="card-text fw-bold">@Model.Price.ToString("C")</p>
    </div>
    <div class="card-footer">
        <a asp-controller="Products" asp-action="Details" asp-route-id="@Model.Id"
           class="btn btn-primary btn-sm">
            View Details
        </a>
    </div>
</div>
```

> [!ad-note] Key Principle
> A partial view is purely a rendering concern. It takes data that has already been prepared and turns it into HTML. It does NOT fetch data from a database, call services, or perform business logic. If you need that, use a [[#View Components|view component]].

> [!summary] Section Summary
> - Partial views are reusable `.cshtml` fragments embedded within other views
> - They are presentation-only: they render data, they do not fetch it
> - Common for cards, form fragments, navigation menus, and repeated UI patterns
> - Named with an underscore prefix by convention (`_ProductCard.cshtml`)

---
