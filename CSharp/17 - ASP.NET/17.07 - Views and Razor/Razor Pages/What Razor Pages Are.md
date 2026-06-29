---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


**Razor Pages** is a server-side, page-centric framework built on top of ASP.NET Core MVC. Each "page" is a pair of files:

1. `Index.cshtml` -- the Razor view with HTML and [[Razor Syntax]]
2. `Index.cshtml.cs` -- the **PageModel** class (the code-behind) containing handler methods and properties

The two files together form a self-contained unit that handles requests and renders responses for a specific URL.

```
/Pages/
    /Products/
        Index.cshtml        <- the page (Razor view)
        Index.cshtml.cs     <- the PageModel (C# code-behind)
        Create.cshtml
        Create.cshtml.cs
        Edit.cshtml
        Edit.cshtml.cs
        Delete.cshtml
        Delete.cshtml.cs
    Index.cshtml
    Index.cshtml.cs
    Privacy.cshtml
    Privacy.cshtml.cs
    _ViewImports.cshtml
    _ViewStart.cshtml
```

Razor Pages was introduced in ASP.NET Core 2.0. It is not a replacement for MVC -- both coexist in the same application. In fact, Razor Pages runs on top of the MVC infrastructure (routing, model binding, filters, etc.).

> [!ad-note] Not Razor Components
> Do not confuse **Razor Pages** (server-rendered, page-focused MVC) with **Razor Components** (Blazor's interactive component model). They share [[Razor Syntax]] but serve very different purposes.

> [!summary] Section Summary
> - Razor Pages pairs a `.cshtml` view with a `.cshtml.cs` PageModel code-behind
> - It is built on top of ASP.NET Core MVC infrastructure
> - Pages are self-contained units handling specific URLs
> - Razor Pages coexists with MVC controllers in the same application

---
