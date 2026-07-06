---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


> [!tip] Complete Summary
> **Razor Pages** is a page-focused programming model in ASP.NET Core where each page is a self-contained pair: a `.cshtml` Razor view and a `.cshtml.cs` PageModel code-behind. Pages live in the `/Pages/` directory and use **convention-based routing** where the file path maps directly to the URL (`/Pages/Products/Index.cshtml` -> `/Products`).
>
> The **`@page` directive** (required, must be first) makes a `.cshtml` file a routable Razor Page. The **PageModel** class inherits from `PageModel`, uses constructor injection for services, and exposes public properties that serve as the view model. **Handler methods** follow the `On{Verb}[Async]` pattern: `OnGetAsync()` loads data, `OnPostAsync()` processes forms. **Named handlers** (`OnPostDeleteAsync()`) support multiple POST actions on a single page, triggered via `asp-page-handler`.
>
> **`[BindProperty]`** marks properties for automatic model binding from form data (POST by default; `SupportsGet = true` for GET). This is the most critical attribute in Razor Pages -- forgetting it results in null properties. **Route parameters** are defined in the `@page` directive (`@page "{id:int}"`) and support all standard constraints.
>
> Razor Pages shares the same [[Layouts and Sections|layout system]], [[Partial Views and View Components|partial views and view components]], [[Tag Helpers]], and validation infrastructure as MVC. The two models coexist naturally in the same application. Razor Pages excels at **form-heavy CRUD pages** and **admin panels**, while MVC controllers are better suited for **complex routing, APIs, and multi-view workflows**. The choice is per-feature, not per-application.

---
