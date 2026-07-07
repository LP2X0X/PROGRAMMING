---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


Since `@` is the Razor transition character, you need to escape it when you want a literal `@` in the output:

```cshtml
@* Double @@ produces a single literal @ *@
<p>Email: user@@example.com</p>

@* Output: Email: user@example.com *@
```

> [!ad-note] Smart Parsing
> Razor is intelligent about email addresses. If you write `user@example.com` in HTML context, Razor recognizes it as an email (not a C# expression) and outputs it literally. You typically only need `@@` when the context is ambiguous -- for example, inside attribute values or when the text after `@` looks like a valid C# identifier.

Use `@@` in these situations:
- Inside CSS `@@media` queries written inline (rare, but happens)
- Twitter/social handles: `@@username`
- Any context where Razor might misinterpret `@` followed by an identifier

> [!summary] Section Summary
> - `@@` produces a literal `@` in the output
> - Razor auto-detects email addresses and does not try to parse them
> - Use `@@` when the context is ambiguous or when `@` is followed by a valid C# identifier

---
