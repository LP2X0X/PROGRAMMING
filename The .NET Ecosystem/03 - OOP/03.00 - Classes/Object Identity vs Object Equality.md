---
tags:
 - csharp
 - oop
 - keyword
---

## Object Identity (Reference Equality)

Two variables are **identical** when they point to the same object in memory — the same heap address.

```csharp
var a = new Person("Long");
var b = a;

// true — both reference the exact same object
Console.WriteLine(ReferenceEquals(a, b));
```

Check identity with:
- `Object.ReferenceEquals(a, b)`
- `==` on reference types (default behavior, unless overloaded)

## Object Equality (Value Equality)

Two objects are **equal** when they contain the same state, even if they are separate instances.

```csharp
var a = new Person("Long");
var b = new Person("Long");

// false — different objects in memory
Console.WriteLine(ReferenceEquals(a, b));

// true — if Equals is overridden to compare by name
Console.WriteLine(a.Equals(b));
```

Check equality with:
- `a.Equals(b)` (override `Equals` to define what "same state" means)
- `==` (if the operator is overloaded)

## Key Differences

|                             | Identity                   | Equality                           |
| --------------------------- | -------------------------- | ---------------------------------- |
| Checks                      | Same memory address        | Same state/values                  |
| Default for reference types | `==` and `ReferenceEquals` | Must override `Equals`             |
| Default for value types     | N/A (no reference)         | Compares all fields via reflection |

## When Overriding `Equals`

If you override `Equals`, you should also:
- Override `GetHashCode` — equal objects must return the same hash code, otherwise dictionaries and hash sets break.
- Consider overloading `==` and `!=` for consistency.

```csharp
public class Person
{
    public string Name { get; }

    public Person(string name) => Name = name;

    public override bool Equals(object obj) =>
        obj is Person other && Name == other.Name;

    public override int GetHashCode() => Name.GetHashCode();
}
```

## Records: Equality Built In

C# `record` types generate value-based equality automatically — no need to override anything.

```csharp
public record Person(string Name);

var a = new Person("Long");
var b = new Person("Long");

Console.WriteLine(a == b);              // true — value equality
Console.WriteLine(ReferenceEquals(a, b)); // false — different instances
```
