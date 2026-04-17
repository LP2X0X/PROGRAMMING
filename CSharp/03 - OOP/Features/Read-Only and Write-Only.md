---
tags:
 - csharp
 - oop
 - encapsulation
---

C# offers several ways to restrict read or write access to data, across both fields and properties. Each serves a different scenario depending on when and where the value should be settable.

## Fields

### `readonly` — set at declaration or in the constructor, then frozen

```csharp
public class Order
{
    private readonly string _id;
    private readonly DateTime _created = DateTime.UtcNow;

    public Order(string id) => _id = id;

    public void Update()
    {
        // _id = "new";  // compile error — readonly after construction
    }
}
```

### `const` — compile-time constant, implicitly static

```csharp
public class MathHelper
{
    private const double Pi = 3.14159;
    public const int MaxRetries = 3;

    // const values are baked into the calling assembly at compile time
}
```

### `readonly` vs `const`

| | `readonly` | `const` |
|---|---|---|
| Set at | Declaration or constructor | Declaration only |
| Evaluated at | Runtime | Compile time |
| Can be `static`? | Yes (optionally) | Always static (implicitly) |
| Can use `new`? | Yes (`readonly List<int> _items = new()`) | No — must be a compile-time literal |
| Types allowed | Any type | Primitives, `string`, `null` |

```ad-warning
There is no write-only field concept in C#. A field you can set but never read would serve no purpose.
```

---

## Properties

### Read-only — no setter at all (`{ get; }`)

Can only be set in the constructor. Fully immutable after construction.

```csharp
public string Id { get; }

public Person(string id) => Id = id;
// Id can never change after this
```

### Read-only externally, writable internally (`{ get; private set; }`)

External code can read but not write. The class itself can modify it anytime.

```csharp
public string Name { get; private set; }

public void Rename(string name) => Name = name;  // works inside the class
```

### Init-only (C# 9+) (`{ get; init; }`)

Can be set during object initialization only — in the constructor or an object initializer. After that, it's frozen.

```csharp
public string Email { get; init; }

var p = new Person { Email = "a@b.com" };  // works
// p.Email = "new@b.com";  // compile error — init window has closed
```

### Write-only (`{ set; }`)

No getter — the value goes in but can never be read back. Rare, but valid for sensitive data.

```csharp
private string _hash;

public string Password
{
    set => _hash = HashAlgorithm.ComputeHash(value);
}
```

### Full read-write (`{ get; set; }`)

No restrictions — anyone can read and write.

```csharp
public string Name { get; set; }
```

---

## Summary

| Pattern | Field | Property |
|---|---|---|
| **Full read-write** | `private int _x;` | `{ get; set; }` |
| **Read-only (immutable)** | `readonly int _x;` | `{ get; }` |
| **Read-only externally** | N/A | `{ get; private set; }` |
| **Set once at init** | `readonly int _x;` | `{ get; init; }` |
| **Compile-time constant** | `const int X = 5;` | N/A |
| **Write-only** | N/A | `{ set; }` (no getter) |

---

## `{ get; }` vs `{ get; private set; }` vs `{ get; init; }`

These three look similar but differ in **when and where** the value can change:

```csharp
public class Person
{
    public string Id { get; }                // constructor only, then frozen forever
    public string Name { get; private set; } // class can change anytime internally
    public string Email { get; init; }       // constructor or object initializer only

    public Person(string id) => Id = id;

    public void Rename(string name)
    {
        Name = name;     // works — private set allows internal writes
        // Id = "new";   // compile error — get-only, frozen after constructor
    }
}

// Caller side:
var p = new Person("001") { Email = "a@b.com" };
// p.Name = "X";          // compile error — private set
// p.Email = "new@b.com"; // compile error — init window closed
```

| | `{ get; }` | `{ get; private set; }` | `{ get; init; }` |
|---|---|---|---|
| Constructor | Yes | Yes | Yes |
| Object initializer | No | No | Yes |
| Inside the class (after construction) | No | Yes | No |
| Outside the class | No | No | No |
