---
tags:
 - csharp
 - oop
 - abstraction
---

# Role of Abstraction

- Abstraction is the pillar of OOP concerned with **exposing only what is relevant** and **suppressing unnecessary detail**. While encapsulation hides *how* something works, abstraction decides *what the caller needs to know* in the first place.
- In C#, abstraction is expressed through **interfaces**, **abstract classes**, and well-designed public APIs. The goal is to let callers program against a simplified model without being coupled to concrete implementations.

---

## Why Abstraction Exists

1. **Simplified mental model** — callers think in terms of *what* an object does, not *how* it does it. A `Stream` is something you read from or write to — you don't need to know whether it wraps a file, a network socket, or a memory buffer.
2. **Loose coupling** — code that depends on an abstraction can work with any concrete type that fulfills the contract, without being rewritten or recompiled.
3. **Substitutability** — you can swap implementations behind an abstraction. Today it's a local cache, tomorrow it's Redis — the callers don't change.

```csharp
// Without abstraction — caller is coupled to a specific type
SqlConnection conn = new SqlConnection(connString);
conn.Open();

// With abstraction — caller depends on the contract, not the implementation
IDbConnection conn = GetConnection();  // could be SQL, Postgres, SQLite...
conn.Open();
```

---

## Abstract Classes

- A class marked `abstract` **cannot be instantiated** directly — it exists only to be a base class.
- It can contain **abstract members** (no implementation — derived classes *must* override them) and **concrete members** (with implementation — inherited as-is).
- Use abstract classes when derived types share common state or behavior.

```csharp
abstract class Shape
{
    public string Color { get; set; }               // concrete — shared state
    public abstract double GetArea();                // abstract — each shape calculates differently
    public void Describe()                           // concrete — shared behavior
        => Console.WriteLine($"{Color} shape, area = {GetArea():F2}");
}

class Circle : Shape
{
    public double Radius { get; set; }
    public override double GetArea() => Math.PI * Radius * Radius;
}

class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }
    public override double GetArea() => Width * Height;
}

// Shape s = new Shape();  // compile error — cannot instantiate abstract class
Shape s = new Circle { Color = "Red", Radius = 5 };
s.Describe();  // "Red shape, area = 78.54"
```

```ad-note
An abstract class can have constructors. They cannot be called directly, but derived classes invoke them through `base(...)`. This lets the abstract class enforce initialization of shared state.
```

---

## Interfaces

- An interface defines a **contract** — a set of members that implementing types must provide. It contains no state and (before C# 8) no implementation.
- A class can implement **multiple interfaces**, unlike class inheritance which is limited to a single base class.

```csharp
interface IRenderable
{
    void Render();
}

interface ISerializable
{
    byte[] Serialize();
}

class Sprite : IRenderable, ISerializable
{
    public void Render() => Console.WriteLine("Drawing sprite");
    public byte[] Serialize() => Encoding.UTF8.GetBytes("sprite-data");
}
```

### Default Interface Methods (C# 8+)

- Starting with C# 8, interfaces can provide **default implementations**. Implementing classes inherit the default unless they override it.
- This allows adding new members to an existing interface without breaking all implementors.

```csharp
interface ILogger
{
    void Log(string message);

    // Default implementation — implementors don't have to override this
    void LogError(string message) => Log($"ERROR: {message}");
}
```

```ad-warning
Default interface methods cannot access instance state (there are no fields in interfaces). They can only call other interface members. If you need shared state + shared behavior, use an abstract class instead.
```

---

## Abstract Class vs Interface

|                              | Abstract Class                              | Interface                                 |
| ---------------------------- | ------------------------------------------- | ----------------------------------------- |
| Can have fields/state?       | Yes                                         | No                                        |
| Can have constructors?       | Yes                                         | No                                        |
| Can have concrete methods?   | Yes                                         | Yes (C# 8+ default methods)               |
| Multiple inheritance?        | No — single base class only                 | Yes — a class can implement many          |
| Access modifiers on members? | Any (`private`, `protected`, etc.)          | `public` by default (C# 8+ allows others) |
| When to use                  | Related types that share state and behavior | Unrelated types that share a capability   |

```ad-tip
Ask yourself: **"Is this a thing the type *is*, or a thing the type *can do*?"**
- "Is a" → abstract class (`Employee` *is a* `Person`)
- "Can do" → interface (`Employee` *can do* `ISerializable`, `IComparable`)
```

---

## Levels of Abstraction

Abstraction is not just interfaces and abstract classes — it operates at every level of design:

| Level         | Example          | What's hidden                                                       |
| ------------- | ---------------- | ------------------------------------------------------------------- |
| **Method**    | `list.Sort()`    | The sorting algorithm (quicksort, introsort, etc.)                  | 
| **Class**     | `HttpClient`     | Socket management, connection pooling, TLS handshake                |
| **Interface** | `IEnumerable<T>` | Whether the data is in a list, array, database, or generated lazily |
| **Assembly**  | A NuGet package  | Entire subsystem internals — callers only see the public API        |

Good abstraction means each level exposes **just enough** for the level above to do its job.

---

## Abstraction vs Encapsulation

These two pillars are closely related but solve different problems:

|          | Abstraction                                            | Encapsulation                                       |
| -------- | ------------------------------------------------------ | --------------------------------------------------- |
| Focus    | *What* to expose                                       | *How* to hide the rest                              |
| Level    | Design-level (interfaces, APIs)                        | Implementation-level (access modifiers, properties) | 
| Question | "What does the caller need to know?"                   | "Who can access this?"                              |
| Tools    | Interfaces, abstract classes, public method signatures | `private`, `protected`, properties, methods         |

```ad-note
A class can be well-encapsulated (all fields private) but poorly abstracted (exposes 50 public methods when the caller only needs 3). Encapsulation protects the internals; abstraction simplifies the surface. They work together but are not the same thing.
```

---

## Common Pitfalls

- **Leaky abstraction** — when implementation details bleed through the abstract surface. Example: an `IRepository` interface that exposes `IQueryable<T>`, leaking the LINQ-to-SQL query engine to callers.
- **Premature abstraction** — introducing interfaces and abstract classes before there are multiple implementations. If there's only one `IFoo` and one `Foo`, the interface adds indirection without benefit.
- **Too-wide interfaces** — an interface with 10 methods forces implementors to provide all 10, even if they only need 2. Prefer small, focused interfaces (Interface Segregation Principle).

```csharp
// Too wide — not every shape can be resized or serialized
interface IShape
{
    double GetArea();
    void Resize(double factor);
    byte[] Serialize();
}

// Better — split by capability
interface IShape { double GetArea(); }
interface IResizable { void Resize(double factor); }
interface ISerializable { byte[] Serialize(); }
```
