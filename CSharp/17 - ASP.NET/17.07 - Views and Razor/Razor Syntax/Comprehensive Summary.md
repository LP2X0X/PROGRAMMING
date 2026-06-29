---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


> [!tip] Complete Summary
> **Razor** is ASP.NET Core's server-side markup syntax that uses `@` to embed C# into HTML. It comes in two expression forms: **implicit** (`@Model.Name` for simple property chains) and **explicit** (`@(expression)` for complex calculations, ternary operators, and disambiguation). **Code blocks** (`@{ }`) allow multi-line C# logic, while **control flow** keywords (`@if`, `@foreach`, `@switch`, `@for`) let you conditionally render HTML. When outputting plain text inside code blocks, use `@:` for single lines or `<text>` for multiple lines.
>
> Razor's **automatic HTML encoding** is a critical security feature that prevents XSS attacks -- all output is encoded by default, and `@Html.Raw()` should only be used with sanitized, trusted content. **Comments** (`@* *@`) are stripped entirely from output, unlike HTML comments.
>
> **Directives** configure the view: `@model` declares the strongly-typed model (enabling IntelliSense), `@using` imports namespaces, `@inject` provides DI access, and `@addTagHelper` enables tag helpers. The `@functions` directive allows defining helper methods in the view, though this should be used sparingly.
>
> Two special shared files govern behavior across views: **`_ViewImports.cshtml`** shares directives (imports, tag helpers) across all views in its directory tree, while **`_ViewStart.cshtml`** runs code before each view (typically setting the default layout). Both cascade downward through subdirectories. Finally, `@@` produces a literal `@` character, though Razor is smart enough to auto-detect email patterns.

---
