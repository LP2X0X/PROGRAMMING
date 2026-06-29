---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


**Explicit expressions** use `@(...)` to clearly delineate where the C# expression starts and ends. Use them whenever the expression is more complex than a simple property chain.

```cshtml
@* Arithmetic *@
<p>Price with tax: @(Model.Price * 1.1)</p>

@* String concatenation *@
<p>Full name: @(Model.FirstName + " " + Model.LastName)</p>

@* Ternary operator *@
<p class="@(Model.IsActive ? "active" : "inactive")">Status</p>

@* Disambiguation from email addresses *@
<p>Contact: support@(company).com</p>

@* Generic types *@
<p>@(new List<string> { "A", "B", "C" }.Count)</p>
```

> [!tip] Practical Tip
> One of the most common uses for explicit expressions is embedding C# in HTML attributes. Without parentheses, Razor might misinterpret where the expression ends:
> ```cshtml
> @* Wrong -- Razor sees @Model.CssClass as the whole attribute value *@
> <div class="container @Model.CssClass-modifier">
>
> @* Correct -- explicit boundary *@
> <div class="container @(Model.CssClass)-modifier">
> ```

**The email problem** is a classic gotcha. Razor is smart enough to recognize email patterns and will NOT try to parse `user@example.com` as a Razor expression. But if you genuinely want to insert a C# expression that looks like an email, use explicit syntax: `user@(domainVariable).com`.

> [!summary] Section Summary
> - Explicit expressions use `@(...)` to wrap complex C# expressions
> - Required for arithmetic, ternary operators, string concatenation, and generics
> - Essential for disambiguating expressions inside HTML attributes
> - Razor auto-detects email patterns, but explicit syntax overrides this when needed

---
