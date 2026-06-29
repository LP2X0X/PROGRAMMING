---
tags:
  - csharp
  - asp-net-core
  - routing
  - constraints
---


ASP.NET Core provides a comprehensive set of built-in constraints. Here is the complete reference.

### Type Constraints

These constrain a parameter to a specific data type:

| Constraint | Description | Example | Matches | Does Not Match |
|---|---|---|---|---|
| `int` | 32-bit integer | `{id:int}` | `123`, `-5` | `abc`, `3.14` |
| `long` | 64-bit integer | `{id:long}` | `9223372036854775807` | `abc` |
| `bool` | Boolean | `{active:bool}` | `true`, `false` | `yes`, `1` |
| `decimal` | Decimal number | `{price:decimal}` | `49.99`, `100` | `abc` |
| `double` | Double-precision float | `{lat:double}` | `47.6062`, `-122.33` | `abc` |
| `float` | Single-precision float | `{val:float}` | `3.14` | `abc` |
| `guid` | GUID | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638` | `not-a-guid` |
| `datetime` | DateTime value | `{date:datetime}` | `2026-06-18`, `06/18/2026` | `not-a-date` |

> [!warning] Common Misconception
> The `bool` constraint only matches the literal strings `"true"` and `"false"` (case-insensitive). It does **not** match `"1"`, `"0"`, `"yes"`, or `"no"`. If you need those values, use a custom constraint or handle conversion in the action.

### String Length Constraints

| Constraint | Description | Example | Matches | Does Not Match |
|---|---|---|---|---|
| `minlength(n)` | Minimum string length | `{name:minlength(3)}` | `abc`, `abcdef` | `ab` |
| `maxlength(n)` | Maximum string length | `{name:maxlength(50)}` | `short` | 51+ char string |
| `length(n)` | Exact string length | `{code:length(6)}` | `ABC123` | `ABC` |
| `length(min,max)` | Length range | `{code:length(3,10)}` | `ABC`, `ABCDEFGHIJ` | `AB` |

### Numeric Range Constraints

| Constraint | Description | Example | Matches | Does Not Match |
|---|---|---|---|---|
| `min(n)` | Minimum value | `{age:min(18)}` | `18`, `99` | `17`, `0` |
| `max(n)` | Maximum value | `{quantity:max(999)}` | `1`, `999` | `1000` |
| `range(min,max)` | Value within range | `{age:range(18,120)}` | `18`, `65`, `120` | `17`, `121` |

### Content Constraints

| Constraint | Description | Example | Matches | Does Not Match |
|---|---|---|---|---|
| `alpha` | Alphabetic characters only | `{name:alpha}` | `John`, `abc` | `John123`, `a b` |
| `regex(expr)` | Matches a regex pattern | `{slug:regex(^[a-z0-9-]+$)}` | `my-product` | `My Product!` |
| `required` | Must have a value (not empty) | `{name:required}` | `anything` | (empty string) |

### Special Constraints

| Constraint | Description | Example |
|---|---|---|
| `exists` | Checks if the value matches a known area, controller, etc. | `{area:exists}` |
| `nonfile` | Matches when the value does not look like a filename (no extension) | `{path:nonfile}` |

> [!tip] Practical Tip
> The `regex` constraint is powerful but has a syntax quirk: because route templates use curly braces, you must **double-escape** braces inside regex patterns or avoid them. Also, in attribute routes, square brackets in regex need to be doubled: `[[a-z]]` instead of `[a-z]` because C# attributes use `[` for their own syntax. In `MapControllerRoute()` string patterns, single `[a-z]` is fine.

> [!example] Regex Constraint in Attribute Routing
> ```csharp
> // Note the doubled brackets [[ ]] in attribute routing
> [HttpGet("posts/{slug:regex(^[[a-z0-9-]]+$)}")]
> public IActionResult GetBySlug(string slug) => Ok();
> ```
>
> In conventional route registration (string-based):
> ```csharp
> // Single brackets are fine here
> app.MapControllerRoute(
>     name: "post",
>     pattern: "posts/{slug:regex(^[a-z0-9-]+$)}");
> ```

> [!summary] Section Summary
> - Built-in constraints cover types (`int`, `guid`, `bool`, `datetime`), string lengths, numeric ranges, content (`alpha`, `regex`), and special cases (`exists`, `required`).
> - `bool` only matches `"true"`/`"false"`, not `"1"`/`"0"`.
> - Regex constraints in attribute routes require doubled square brackets `[[ ]]`.
> - The `regex` constraint should be used sparingly; prefer named constraints when possible.
