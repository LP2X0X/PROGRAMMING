---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


**Razor** is a server-side markup syntax developed by Microsoft that allows you to embed C# code directly into HTML files. The Razor engine parses `.cshtml` files (or `.razor` for Blazor components) and compiles them into C# classes that produce HTML output at runtime.

The fundamental concept is the **`@` transition character**. The Razor parser uses `@` to switch from HTML mode to C# mode. The parser is smart enough to figure out where the C# expression ends and HTML begins again, which is what makes Razor feel natural compared to older syntaxes like Web Forms (`<%= %>`).

Razor files in ASP.NET Core MVC live under the `/Views/` folder, organized by controller:

```
/Views/
    /Home/
        Index.cshtml
        About.cshtml
    /Products/
        Index.cshtml
        Details.cshtml
    /Shared/
        _Layout.cshtml
        _ValidationScriptsPartial.cshtml
    _ViewImports.cshtml
    _ViewStart.cshtml
```

> [!ad-note] Razor vs Blazor Components
> Razor syntax is used in two distinct contexts: **Razor Views** (`.cshtml`, server-rendered MVC/Razor Pages) and **Razor Components** (`.razor`, Blazor). While they share the same core syntax, Blazor components add event handling, component parameters, and an interactive rendering model. This note focuses on Razor in the MVC/Razor Pages context.

> [!summary] Section Summary
> - Razor is a server-side markup syntax using `@` to transition between HTML and C#
> - Files use the `.cshtml` extension and compile to C# classes at runtime
> - Views are organized under `/Views/` by controller name
> - Razor is distinct from Blazor's `.razor` components, though they share syntax foundations

---
