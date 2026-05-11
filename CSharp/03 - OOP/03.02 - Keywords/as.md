---
tags:
 - csharp
 - oop
 - keyword
---

# The `as` Keyword

- The `as` keyword performs a **safe cast** — it attempts to convert an object to a specified type and returns `null` if the cast fails, instead of throwing an `InvalidCastException`.
- It is the safe alternative to an explicit cast `(Type)obj`.

---

## Syntax

```csharp
expression as Type
```

- If `expression` is compatible with `Type` → returns the casted object.
- If `expression` is NOT compatible → returns `null` (no exception).

---

## Basic Example

```csharp
object obj = "Hello, World!";

// Safe cast with "as"
string? str = obj as string;

if (str != null)
{
    Console.WriteLine(str.Length); // 13
}

// Failed cast — returns null, no exception
int? num = obj as int?; // null — a string is not an int
```

---

## `as` vs Explicit Cast `(Type)`

```csharp
object obj = "Hello";

// Explicit cast — throws if it fails
string s1 = (string)obj;      // works
// int n = (int)obj;           // throws InvalidCastException

// "as" — returns null if it fails
string? s2 = obj as string;   // works
int? n2 = obj as int?;        // null — no exception
```

| | Explicit cast `(T)` | `as` |
|---|---|---|
| On success | Returns the casted object | Returns the casted object |
| On failure | Throws `InvalidCastException` | Returns `null` |
| Works with value types | Yes | Only nullable value types (`int?`, not `int`) |
| Works with reference types | Yes | Yes |
| Performance on failure | Expensive (exception) | Cheap (just returns null) |

```ad-important
`as` can only be used with **reference types** and **nullable value types**. You cannot write `obj as int` because `int` cannot be `null`. Use `obj as int?` instead.
```

---

## `as` vs `is`

Both are used for safe type checking, but they work differently:

### `as` — cast first, check null after

```csharp
object obj = "Hello";

string? str = obj as string;
if (str != null)
{
    Console.WriteLine(str.Length);
}
```

### `is` with pattern matching (C# 7+) — check and cast in one step

```csharp
object obj = "Hello";

if (obj is string str)
{
    Console.WriteLine(str.Length); // str is already cast
}
```

| | `as` | `is` with pattern matching |
|---|---|---|
| Steps | cast, then null-check | check + cast in one step |
| Result on failure | `null` | does not enter the `if` block |
| Variable scope | available after the `as` line (could be null) | only available inside the `if` block (never null) |
| Works with value types | only nullable (`int?`) | yes (`int`, `int?`, any type) |
| Readability | two lines | one line |

```ad-tip
In modern C# (7+), **prefer `is` with pattern matching** over `as` in most cases. It combines the check and cast into one step, and the variable is guaranteed non-null inside the block. Use `as` when you need the variable to exist outside the `if` scope, or when you want to work with a potentially-null result.
```

---

## Common Patterns

### Safely handling different types

```csharp
void ProcessMessage(object message)
{
    var email = message as EmailMessage;
    if (email != null)
    {
        SendEmail(email);
        return;
    }

    var sms = message as SmsMessage;
    if (sms != null)
    {
        SendSms(sms);
        return;
    }

    Console.WriteLine("Unknown message type");
}
```

```ad-note
The above works, but in modern C# a `switch` with pattern matching is cleaner:
```

```csharp
void ProcessMessage(object message)
{
    switch (message)
    {
        case EmailMessage email:
            SendEmail(email);
            break;
        case SmsMessage sms:
            SendSms(sms);
            break;
        default:
            Console.WriteLine("Unknown message type");
            break;
    }
}
```

### Null-conditional chaining with `as`

```csharp
// Try to cast and immediately access a member — one line
int? length = (obj as string)?.Length;
// If obj is not a string → null
// If obj is a string    → its length
```

### Event handler pattern

```csharp
void OnClick(object sender, EventArgs e)
{
    var button = sender as Button;
    if (button != null)
    {
        button.Text = "Clicked!";
    }
}
```

---

## With Inheritance Hierarchies

- `as` respects the inheritance chain — it succeeds for any type in the hierarchy:

```csharp
class Animal { }
class Dog : Animal { }
class Cat : Animal { }

Dog dog = new Dog();

Animal? a = dog as Animal;  // succeeds — Dog is an Animal
Cat? c = dog as Cat;        // null — Dog is not a Cat
object? o = dog as object;  // succeeds — everything is an Object
```

---

## When `as` Returns `null`

The cast fails and returns `null` in these cases:
1. The object is of an **incompatible type**
2. The object is **already `null`** (null as anything is null)
3. There is **no implicit or explicit conversion** defined between the types

```csharp
string? s1 = null as string;           // null — null stays null
string? s2 = 42 as string;             // compile error — int is never string
string? s3 = (object)42 as string;     // null at runtime — boxed int is not string
```

```ad-warning
`as` only works for **reference conversions** and **nullable conversions**. It does NOT call user-defined conversion operators (`implicit`/`explicit`). If you need custom conversions, use an explicit cast instead.
```

---

## Summary

| Scenario | Use |
|---|---|
| You need a safe cast with null fallback | `as` |
| You need a safe cast inside a condition | `is` with pattern matching (preferred) |
| You are certain the cast will succeed | explicit cast `(T)` |
| You need to check type without casting | `is` (without pattern variable) |
| Multiple type checks | `switch` with pattern matching |
