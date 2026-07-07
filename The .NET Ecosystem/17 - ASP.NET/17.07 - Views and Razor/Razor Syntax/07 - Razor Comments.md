---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


Razor has its own comment syntax that is stripped from the output entirely (unlike HTML comments which are visible in the page source):

```cshtml
@* This is a Razor comment.
   It can span multiple lines.
   It is NOT sent to the browser. *@

<!-- This is an HTML comment. It IS visible in page source. -->
```

> [!tip] Practical Tip
> Use Razor comments (`@* *@`) for development notes and TODOs. Use HTML comments (`<!-- -->`) only when you intentionally want the comment visible in the rendered HTML source (rare in production).

You can also use C# comments inside code blocks:

```cshtml
@{
    // Single-line C# comment
    var name = Model.Name;

    /* Multi-line
       C# comment */
    var price = Model.Price;
}
```

> [!summary] Section Summary
> - Razor comments use `@* ... *@` and are completely stripped from the output
> - HTML comments `<!-- -->` are sent to the browser and visible in page source
> - Standard C# comments (`//` and `/* */`) work inside code blocks
> - Prefer Razor comments over HTML comments for development notes

---
