---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


1. **Simpler for CRUD**: a Create page is two files (`Create.cshtml` + `Create.cshtml.cs`), not three (controller + view + view model)
2. **Self-contained**: the page and its logic live together, making the feature easy to find and understand
3. **Convention-based routing**: file path = URL, no route configuration needed
4. **Less boilerplate**: no controller class with multiple actions, no `[HttpGet]`/`[HttpPost]` attributes
5. **Easier for beginners**: the programming model is more intuitive than MVC's indirection
6. **Full MVC power**: still has model binding, validation, filters, DI, tag helpers, etc.
7. **Coexists with MVC**: use Pages for some features, controllers for others

> [!tip] Practical Tip
> The ASP.NET Core project templates use Razor Pages by default for Identity UI (login, register, manage account). Even in MVC-heavy applications, the Identity pages are Razor Pages under `/Areas/Identity/Pages/`.

> [!summary] Section Summary
> - Razor Pages reduces file count and ceremony for page-oriented features
> - Convention-based routing eliminates manual route configuration
> - The self-contained nature (page + model together) improves feature discoverability
> - Full access to MVC infrastructure: model binding, validation, DI, filters, tag helpers

---
