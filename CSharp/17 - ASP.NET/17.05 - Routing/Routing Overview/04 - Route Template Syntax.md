---
tags:
  - csharp
  - asp-net-core
  - routing
  - fundamentals
---


Route templates define the URL patterns that routing matches against. They are strings composed of **literal segments**, **route parameters**, and special syntax.

### Literal Segments

Fixed text that must match exactly:

```
"products/catalog"
```

Matches: `/products/catalog`
Does not match: `/products/list` or `/items/catalog`

### Route Parameters

Placeholders enclosed in curly braces that capture values from the URL:

```
"products/{id}"
```

Matches: `/products/5`, `/products/abc`, `/products/anything-here`
The captured value is available as a route value named `id`.

### Optional Parameters

Append `?` to make a parameter optional:

```
"products/{id?}"
```

Matches: `/products` (id is null) and `/products/5` (id is "5")

### Default Values

Use `=` to specify a default when the segment is absent:

```
"{controller=Home}/{action=Index}/{id?}"
```

| URL | controller | action | id |
|---|---|---|---|
| `/` | `Home` | `Index` | `null` |
| `/Products` | `Products` | `Index` | `null` |
| `/Products/Edit/3` | `Products` | `Edit` | `3` |

### Catch-All Parameters

Prefix with `*` or `**` to capture the rest of the URL (including slashes):

```
"files/{**path}"
```

| URL                            | path                    |
|---|---|
| `/files/docs/readme.txt`       | `docs/readme.txt`       |
| `/files/images/2026/photo.jpg` | `images/2026/photo.jpg` |
| `/files/`                      | `` (empty string)       |

> [!ad-note] Important
> `{*path}` and `{**path}` behave identically. The double-asterisk form is preferred for clarity and is the documented convention.

### Route Constraints

Constraints restrict which values a parameter will match. They are appended with a colon:

```
"products/{id:int}"
```

Matches: `/products/5` (id is an integer)
Does not match: `/products/abc` (not an integer, routing moves to the next route)

See [[Route Constraints and Tokens]] for the full list of built-in constraints.

### Complex Segments

Parameters can be mixed with literals in a single segment:

```
"files/{filename}.{ext}"
```

Matches: `/files/report.pdf` (filename = "report", ext = "pdf")

### Full Syntax Reference Table

| Syntax | Meaning | Example |
|---|---|---|
| `literal` | Exact match | `products` |
| `{param}` | Required parameter | `{id}` |
| `{param?}` | Optional parameter | `{id?}` |
| `{param=default}` | Default value | `{action=Index}` |
| `{**param}` | Catch-all (greedy) | `{**slug}` |
| `{param:constraint}` | Constrained parameter | `{id:int}` |
| `{param:c1:c2}` | Multiple constraints | `{id:int:min(1)}` |

> [!warning] Common Misconception
> Optional parameters and default values are **not** the same thing. `{id?}` means the segment can be missing and `id` will be `null`. `{id=5}` means if the segment is missing, `id` will be `"5"`. You can combine them: `{id:int?}` means optional with a constraint, but no default.

> [!summary] Section Summary
> - Route templates combine literal segments and parameters in curly braces.
> - Parameters can be required, optional (`?`), have defaults (`=value`), or be catch-all (`**`).
> - Constraints restrict which values match a parameter (e.g., `{id:int}`).
> - Complex segments can mix literals and parameters within a single URL segment.
> - Catch-all parameters capture everything including forward slashes.
