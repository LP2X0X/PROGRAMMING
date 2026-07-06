---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


**Implicit expressions** are the simplest form of Razor syntax. You prefix a C# expression with `@` and Razor automatically determines where the expression ends by following C# identifier rules (letters, digits, dots, brackets).

```cshtml
<h1>@Model.ProductName</h1>
<p>Current time: @DateTime.Now</p>
<p>Total items: @Model.Items.Count</p>
<p>Category: @Model.Category.Name</p>
```

The Razor parser follows the **dot chain** until it encounters something that is not a valid C# continuation (like a space, HTML tag, or certain punctuation).

> [!warning] Common Misconception
> Many developers assume `@Model.Price + tax` will compute a sum. It will not. Razor sees `@Model.Price` as the expression and renders the rest as literal text. The output would be something like `19.99 + tax`. You need [[#Explicit Expressions]] for compound expressions.

**What works with implicit expressions:**
- Property access: `@Model.Name`
- Method calls: `@DateTime.Now.ToString("yyyy-MM-dd")`
- Indexer access: `@Model.Items[0]`
- Chained members: `@Model.Category.Products.Count`

**What does NOT work:**
- Arithmetic: `@Model.Price * 1.1` -- Razor stops at the space before `*`
- String concatenation with `+`
- Ternary operators
- Anything requiring parentheses around the whole expression

> [!ad-note] C# Keywords as Identifiers
> Implicit expressions cannot start with C# keywords. `@class` would confuse the parser because `class` is a keyword. Use explicit expressions for these: `@(myClass)`.

> [!summary] Section Summary
> - Implicit expressions start with `@` followed by a C# expression
> - The parser follows valid C# identifiers (dots, brackets, method calls)
> - They work for simple property access, method calls, and indexers
> - They do NOT work for arithmetic, ternary operators, or compound expressions
> - Use explicit expressions when Razor cannot determine the expression boundary

---
