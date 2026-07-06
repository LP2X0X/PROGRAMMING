---
tags:
 - csharp
 - basics
---

In C#, the underscore `_` serves two different roles depending on context: a **real language discard** (compiler feature, C# 7+) and a **naming convention** (just a variable named `_`).

## Real Discard — Language Feature (C# 7+)

When `_` appears in `out` parameters, deconstruction, or pattern matching, it is a **true discard**. The compiler does not allocate storage for the value — it is genuinely thrown away.

### Out Parameters

```csharp
// only care about whether the parse succeeded, not the result
if (int.TryParse(input, out _))
{
    Console.WriteLine("Valid number");
}
```

### Tuple Deconstruction

```csharp
(int x, _, int z) = GetCoordinates();
// only x and z are captured — the middle value is discarded
```

### Pattern Matching

```csharp
switch (shape)
{
    case Circle c: HandleCircle(c); break;
    case Rectangle r: HandleRect(r); break;
    case _: HandleUnknown(); break;  // catches everything else
}
```

### Lambda / Anonymous Method Parameters

```csharp
button.Click += (_, _) => Console.WriteLine("Clicked");
// both sender and EventArgs are discarded
```

## Convention Only — Variable Named `_`

When you write `var _` or assign to `_` as a standalone variable, it is **not** a discard. It is a regular variable that happens to be named underscore. The value **is** allocated and stored.

```csharp
var _ = SomeMethod();   // a real variable named '_' — value is stored
_ = SomeMethod();       // reassignment to that variable
```

This is just a naming convention to signal "I don't care about this value."

## How to Tell the Difference

| Context | Real discard? | Storage allocated? |
|---|:-:|:-:|
| `out _` | Yes | No |
| `(x, _) = tuple` | Yes | No |
| `case _:` in pattern matching | Yes | No |
| `(_, _) =>` in lambda params | Yes | No |
| `var _ = Method()` | No — just a variable named `_` | Yes |
| `_ = Method()` (standalone) | No — assignment to a variable | Yes |

**Rule of thumb:** if `_` appears where a declaration would normally go (`out _`, tuple `_`, pattern `_`, lambda parameter `_`), it is the real discard. If you write `var _` or use it as a standalone identifier, it is just a variable name.