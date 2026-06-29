---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


**Code blocks** let you write multi-line C# logic inside a view using `@{ ... }`. Code inside a code block executes but does not produce output unless you explicitly write to the response or switch back to HTML mode.

```cshtml
@{
    var greeting = "Welcome";
    var currentHour = DateTime.Now.Hour;
    string timeOfDay;

    if (currentHour < 12)
    {
        timeOfDay = "morning";
    }
    else if (currentHour < 17)
    {
        timeOfDay = "afternoon";
    }
    else
    {
        timeOfDay = "evening";
    }
}

<h1>@greeting, good @timeOfDay!</h1>
```

Variables declared inside a code block are available for the rest of the view (they compile into the same method). You can have multiple code blocks in a single view, and variables from earlier blocks are accessible in later ones.

> [!warning] Common Misconception
> Code blocks should contain **view logic**, not business logic. If you find yourself writing database queries, complex calculations, or business rules in a code block, that logic belongs in the controller, service layer, or [[Partial Views and View Components|view component]]. Views should focus on presentation.

**Mixing HTML and C# inside code blocks:**

```cshtml
@{
    var products = Model.Products;

    if (!products.Any())
    {
        <p class="empty-state">No products found.</p>
    }
}
```

When you are inside a code block (`@{ }`), Razor switches to C# mode. It recognizes HTML tags and automatically transitions back to HTML mode when it encounters a `<tag>`. This is how you can freely mix HTML into C# logic.

> [!summary] Section Summary
> - Code blocks use `@{ ... }` for multi-line C# statements
> - Variables declared in code blocks are available throughout the rest of the view
> - HTML tags inside code blocks automatically transition Razor back to HTML mode
> - Keep code blocks focused on presentation logic, not business logic

---
