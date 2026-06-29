---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


Choosing between Razor Pages and MVC controllers depends on the nature of the UI:

| Aspect | Razor Pages | MVC Controllers |
|---|---|---|
| **Best for** | Form-heavy pages, CRUD, content sites | Complex routing, APIs, multi-view controllers |
| **Organization** | Page-centric (page + model together) | Feature-centric (controller groups related actions) |
| **File count per feature** | 2 files (`.cshtml` + `.cshtml.cs`) | 3+ files (controller + view + view model) |
| **URL mapping** | File path = URL (convention-based) | Route attributes or conventions |
| **Ceremony** | Low -- minimal boilerplate | Higher -- controller, action, view, routing |
| **Testability** | Good (PageModel is testable) | Good (controller is testable) |
| **Typical app** | Admin panels, settings pages, forms | E-commerce with complex workflows, SPAs, APIs |

> [!tip] Practical Tip
> A common pattern in large applications is to use **Razor Pages for admin/CRUD sections** and **MVC controllers for the public-facing site and APIs**. They coexist naturally. Enable both in `Program.cs`:
> ```csharp
> builder.Services.AddRazorPages();
> builder.Services.AddControllersWithViews();
>
> app.MapRazorPages();
> app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
> ```

> [!summary] Section Summary
> - Razor Pages is best for form-heavy, CRUD, and page-oriented scenarios
> - MVC controllers are better for complex routing, APIs, and multi-view workflows
> - Both can coexist in the same application -- choose per feature, not per application
> - Razor Pages has less ceremony: fewer files and no need for separate routing configuration

---
