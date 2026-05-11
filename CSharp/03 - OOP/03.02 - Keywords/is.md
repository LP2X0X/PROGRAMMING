---
tags:
 - csharp
 - oop
 - keyword
---

# The `is` Keyword

- The `is` keyword performs a **type check** — it tests whether an object is compatible with a given type and returns `true` or `false`.
- Since C# 7, `is` also supports **pattern matching**, allowing you to check the type and cast in a single step. This has made `is` far more powerful than a simple type check.

---

## Basic Type Check (Original Usage)

```csharp
object obj = "Hello";

if (obj is string)
{
    Console.WriteLine("It's a string");
}

if (obj is int)
{
    Console.WriteLine("It's an int"); // does not execute
}
```

- Returns `true` if the object is the specified type, a derived type, or implements the interface.
- Returns `false` if the object is `null` or an incompatible type.

```csharp
object obj = null;
obj is string;   // false — null is never "is" anything
```

---

## Type Pattern (C# 7+) — Check and Cast in One Step

- This is the most common modern usage. It checks the type AND creates a typed variable at the same time:

```csharp
object obj = "Hello";

if (obj is string str)
{
    // str is already cast to string — guaranteed non-null
    Console.WriteLine(str.Length); // 5
}
// str is still in scope here but only definitely assigned inside the if block
```

```ad-tip
This replaces the old two-step pattern of `as` + null check. One line instead of two, and the variable is guaranteed non-null inside the block.
```

### Comparison with the old way

```csharp
// Old way (C# 6 and before):
if (obj is string)
{
    string str = (string)obj;  // redundant cast — we already checked
    Console.WriteLine(str.Length);
}

// Or with "as":
string? str = obj as string;
if (str != null)
{
    Console.WriteLine(str.Length);
}

// Modern way (C# 7+):
if (obj is string str)
{
    Console.WriteLine(str.Length);  // one step, no redundancy
}
```

---

## All Pattern Types Supported by `is`

Since C# 7, the `is` keyword has grown to support many patterns. Here are the most important ones:

### Type Pattern

```csharp
if (obj is string str)         // check type + cast
if (obj is int n)              // works with value types too
if (obj is Employee emp)       // works with any type
```

### Constant Pattern

```csharp
if (obj is null)               // null check
if (obj is 42)                 // specific value check
if (obj is "hello")            // string value check
```

```ad-note
`obj is null` is the safest null check — it always checks for true null, unlike `==` which can be overloaded to do something else. The compiler recommends `is null` over `== null` for this reason.
```

### not Pattern (C# 9+)

```csharp
if (obj is not null)           // not null
if (obj is not string)         // not a string
if (obj is not 0)              // not zero
```

### and / or Patterns (C# 9+)

```csharp
if (num is > 0 and < 100)     // range check
if (num is < 0 or > 100)      // outside range
if (obj is string or int)      // either type
```

### Relational Patterns (C# 9+)

```csharp
if (num is > 0)
if (num is >= 18)
if (num is < 0)
if (num is <= 100)
```

```csharp
// Practical example — combining relational patterns
string GetGrade(int score) => score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    >= 60 => "D",
    _     => "F"
};
```

### Property Pattern (C# 8+)

- Check an object's properties directly in the pattern:

```csharp
if (person is { Name: "Alice", Age: > 18 })
{
    Console.WriteLine("Adult Alice found");
}

// Nested property patterns
if (order is { Customer: { Country: "US" }, Total: > 100 })
{
    Console.WriteLine("Big US order");
}

// C# 10 simplified nested pattern
if (order is { Customer.Country: "US", Total: > 100 })
{
    Console.WriteLine("Big US order");
}
```

### var Pattern

- Always matches — assigns the value to a new variable. Useful inside more complex patterns:

```csharp
if (obj is var x)  // always true, x = obj
{
    Console.WriteLine(x);
}

// Useful in LINQ or complex expressions:
var results = people
    .Where(p => p.GetAge() is var age && age >= 18 && age <= 65);
```

### Tuple Pattern

```csharp
var point = (x: 0, y: 0);

string Describe((int x, int y) point) => point switch
{
    (0, 0) => "Origin",
    (0, _) => "On Y axis",
    (_, 0) => "On X axis",
    _      => "Somewhere else"
};
```

### Discard Pattern `_`

- Matches anything — used as a catch-all:

```csharp
string Classify(object obj) => obj switch
{
    int n when n > 0 => "positive int",
    string s         => $"string: {s}",
    null             => "null",
    _                => "something else"  // catch-all
};
```

---

## `is` in Switch Statements and Expressions

- Pattern matching with `is` is most powerful when used in `switch`:

### Switch statement

```csharp
void Process(object obj)
{
    switch (obj)
    {
        case int n when n > 0:
            Console.WriteLine($"Positive int: {n}");
            break;
        case int n:
            Console.WriteLine($"Non-positive int: {n}");
            break;
        case string { Length: > 10 } s:
            Console.WriteLine($"Long string: {s}");
            break;
        case string s:
            Console.WriteLine($"Short string: {s}");
            break;
        case null:
            Console.WriteLine("null");
            break;
        default:
            Console.WriteLine("Unknown");
            break;
    }
}
```

### Switch expression (C# 8+)

```csharp
string Describe(object obj) => obj switch
{
    int n when n > 0     => $"Positive: {n}",
    int n                => $"Non-positive: {n}",
    string s             => $"String: {s}",
    Person { Age: > 18 } => "Adult person",
    null                 => "null",
    _                    => "Unknown"
};
```

---

## `is` with Inheritance

- `is` respects the full inheritance chain:

```csharp
class Animal { }
class Dog : Animal { }
class Labrador : Dog { }

var lab = new Labrador();

lab is Labrador;  // true
lab is Dog;       // true  — Labrador is a Dog
lab is Animal;    // true  — Labrador is an Animal
lab is object;    // true  — everything is an object

lab is string;    // false — Labrador is not a string
```

### With interfaces

```csharp
interface ISwimmable { }
class Duck : Animal, ISwimmable { }

var duck = new Duck();

duck is ISwimmable;  // true
duck is Animal;      // true
duck is Dog;         // false
```

---

## `is` vs `as` vs Explicit Cast

| | `is` (pattern matching) | [[as]] | Explicit cast `(T)` |
|---|---|---|---|
| On success | Returns `true`, creates typed variable | Returns casted object | Returns casted object |
| On failure | Returns `false` | Returns `null` | Throws `InvalidCastException` |
| Null input | Returns `false` | Returns `null` | Throws or returns null |
| Value types | Yes (`int`, `bool`, etc.) | Only nullable (`int?`) | Yes |
| Typical use | `if (obj is Type x)` | `var x = obj as Type` | `var x = (Type)obj` |

```ad-tip
**Modern C# preference:**
1. **`is` with pattern matching** — the default choice for safe type checks
2. **`as`** — when you need the variable outside the `if` scope or want a null result to pass around
3. **Explicit cast** — when you are certain of the type and want a hard failure if you're wrong
```

---

## `is` vs `typeof` / `GetType()`

```csharp
class Animal { }
class Dog : Animal { }

var dog = new Dog();

// "is" — checks the ENTIRE inheritance chain
dog is Animal;                           // true

// GetType() == typeof() — checks the EXACT runtime type only
dog.GetType() == typeof(Animal);         // false — it's a Dog, not Animal
dog.GetType() == typeof(Dog);            // true  — exact match
```

| | `is` | `GetType() == typeof()` |
|---|---|---|
| Checks base types | Yes | No |
| Checks interfaces | Yes | No |
| Checks exact type only | No | Yes |
| Works with `null` | Returns `false` | Throws `NullReferenceException` |

Use `is` when you want "is this compatible with Type?" Use `GetType() == typeof()` when you want "is this exactly this type and nothing else?"

---

## Summary

| Pattern | Syntax | Since |
|---|---|---|
| Type check only | `obj is string` | C# 1.0 |
| Type + cast | `obj is string s` | C# 7.0 |
| Constant | `obj is null`, `obj is 42` | C# 7.0 |
| not | `obj is not null` | C# 9.0 |
| and / or | `n is > 0 and < 100` | C# 9.0 |
| Relational | `n is >= 18` | C# 9.0 |
| Property | `obj is { Name: "Alice" }` | C# 8.0 |
| Tuple | `point is (0, 0)` | C# 8.0 |
| var | `obj is var x` | C# 7.0 |
| Discard | `_` in switch | C# 7.0 |
