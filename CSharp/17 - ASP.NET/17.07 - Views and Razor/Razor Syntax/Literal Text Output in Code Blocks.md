---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


When you are inside a code block and want to output plain text (not wrapped in an HTML tag), Razor needs a hint that you are switching to content mode. There are two mechanisms: `@:` and `<text>`.

### The @: Operator (Single Line)

`@:` tells Razor "treat the rest of this line as literal content output":

```cshtml
@if (Model.ShowGreeting)
{
    @:Hello, this is plain text output without any HTML tag.
}
```

### The `<text>` Element (Multi-Line)

`<text>` is a pseudo-element recognized by Razor. It is **not rendered** to the browser -- it simply tells Razor to treat its contents as text output:

```cshtml
@for (int i = 0; i < 3; i++)
{
    <text>
        Item number @i
        is being processed.
    </text>
}
```

> [!tip] Practical Tip
> You rarely need `@:` or `<text>` in practice. If your content is wrapped in any HTML tag (even a `<span>`), Razor transitions automatically. These are only needed for outputting raw text inside a code block without any wrapping element.

> [!summary] Section Summary
> - `@:` outputs the rest of the current line as literal content (single line)
> - `<text>` wraps multiple lines of literal content output (multi-line)
> - Both are only needed when outputting text inside a code block without an HTML wrapper
> - If your content is inside any HTML element, Razor transitions automatically

---
