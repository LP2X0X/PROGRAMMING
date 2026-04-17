---
tags:
 - csharp
 - basics
 - types
---

## What Is a Struct

A `struct` is a **value type** that stores its data directly on the stack (or inline inside another object) rather than on the heap. It exists primarily for **performance** — avoiding heap allocation and garbage collection for small, lightweight data.

```csharp
struct Point
{
    public int X;
    public int Y;
}
```

## Struct vs Class — Core Difference

```
Class (reference type)              Struct (value type)

Stack          Heap                 Stack
┌─────────┐   ┌──────────┐         ┌──────────┐
│ p = 0x→ │──>│ X = 10   │         │ X = 10   │
└─────────┘   │ Y = 20   │         │ Y = 20   │
              └──────────┘         └──────────┘

- p holds an address              - the data IS right there
- object lives on the heap        - no heap, no address, no GC
- GC must track and clean it      - dies automatically when scope ends
```

### Assignment: Copy vs Shared Reference

```csharp
// Class — shared reference
class PointClass { public int X; public int Y; }

PointClass a = new PointClass { X = 1, Y = 2 };
PointClass b = a;       // b points to the SAME object
b.X = 99;
Console.WriteLine(a.X); // 99 — a is affected

// Struct — independent copy
struct PointStruct { public int X; public int Y; }

PointStruct a = new PointStruct { X = 1, Y = 2 };
PointStruct b = a;       // b is a COPY of a
b.X = 99;
Console.WriteLine(a.X); // 1 — a is unaffected
```

## Comparison Table

| | Struct | Class |
|---|---|---|
| Type category | Value type | Reference type |
| Stored on | Stack (or inline) | Heap |
| Assignment | Copies the entire value | Copies the reference (address) |
| GC pressure | None | Yes |
| Can be null? | No (unless `Nullable<T>`) | Yes |
| Supports inheritance? | No — only implements interfaces | Yes |
| Can be abstract/sealed? | No (implicitly sealed) | Yes |
| Default constructor | Always exists, zeroes all fields | You define it |
| Parameterless constructor | Allowed since C# 10 (but default still zeroes) | Always allowed |
| Equality | Compares all fields by value (default) | Compares references (default) |

## Nuances and Gotchas

### 1. Boxing — When a Struct Goes to the Heap

When you assign a struct to an `object`, `interface`, or `dynamic`, it gets **boxed** — the runtime copies it to the heap and wraps it in an object. This negates the performance benefit.

```csharp
Point p = new Point { X = 1, Y = 2 };
object o = p;          // boxed — copied to the heap
Point p2 = (Point)o;   // unboxed — copied back to the stack
```

Boxing happens silently in many places:
- Passing a struct to a method that takes `object` or an interface
- Adding a struct to a non-generic collection like `ArrayList`
- Calling `ToString()`, `GetHashCode()`, or `Equals()` if not overridden (the runtime boxes to call the base `object` methods)

### 2. Mutability Pitfall

Mutable structs are a common source of bugs because copies are invisible:

```csharp
struct MutablePoint { public int X; public int Y; }

var points = new List<MutablePoint> { new MutablePoint { X = 1, Y = 2 } };

// This does NOT modify the struct in the list
MutablePoint p = points[0];  // p is a COPY
p.X = 99;                    // modifies the copy
Console.WriteLine(points[0].X); // still 1

// This is a compile error — the indexer returns a copy, not a reference
// points[0].X = 99;  // error: cannot modify the return value
```

This is why **structs should be immutable** — use `readonly struct` to enforce it:

```csharp
readonly struct Point
{
    public int X { get; init; }
    public int Y { get; init; }
}
```

### 3. readonly struct

`readonly struct` (C# 7.2) guarantees that **no member can modify state** after construction. The compiler enforces this at compile time.

```csharp
readonly struct Vector2D
{
    public double X { get; }
    public double Y { get; }

    public Vector2D(double x, double y) => (X, Y) = (x, y);
    public double Length => Math.Sqrt(X * X + Y * Y);
}
```

Benefits:
- Prevents the mutability pitfall above
- Enables compiler optimizations — can pass by `in` reference without defensive copies

### 3b. readonly Members on Non-readonly Structs

If you don't want the **entire** struct to be readonly, you can mark **individual methods and properties** as `readonly` instead. This promises that specific member will not modify any field.

```csharp
struct Point
{
    public int X;
    public int Y;

    public readonly double Length => Math.Sqrt(X * X + Y * Y);  // only reads

    public readonly void Print()
    {
        // X = 0;  // compile error — cannot modify fields in a readonly method
        Console.WriteLine($"({X}, {Y})");
    }

    public void Reset()  // non-readonly — can mutate
    {
        X = 0;
        Y = 0;
    }
}
```

**Why it matters — defensive copies:** When a struct is passed with `in` (read-only reference), the compiler must guarantee no mutation. If you call a method that is **not** marked `readonly`, the compiler makes a **defensive copy** of the entire struct before calling it — just in case it modifies something:

```csharp
void Process(in Point p)
{
    // Length IS readonly → called directly on p (no copy)
    var len = p.Length;

    // Reset is NOT readonly → compiler copies p into a temp, calls Reset on the temp
    // (the copy is silently thrown away — the call has no effect)
}
```

Marking methods `readonly` is a **performance hint** that eliminates these hidden copies.

**Applies to:** instance methods, properties, indexers, and overrides like `ToString()` / `GetHashCode()` / `Equals()`. Does not apply to classes — only structs have the defensive copy problem.

### 4. ref struct — Stack-Only Structs

`ref struct` (C# 7.2) restricts a struct to **only ever live on the stack**. It can never be boxed, stored in a class field, or captured by a lambda/async method.

```csharp
ref struct StackOnlyBuffer
{
    public Span<byte> Data;
}
```

`Span<T>` and `ReadOnlySpan<T>` are the most common `ref struct` types. This is used for high-performance, zero-allocation memory access.

Restrictions:
- Cannot be a field in a class or non-ref struct
- Cannot be boxed to `object` or interface
- Cannot be used in `async` methods or captured by lambdas
- Cannot implement interfaces

### 5. record struct (C# 10)

`record struct` gives you value-based equality, `ToString()`, and deconstruction for free — like `record class` but as a value type:

```csharp
record struct Point(int X, int Y);

var a = new Point(1, 2);
var b = new Point(1, 2);
Console.WriteLine(a == b);    // true — value equality, not reference
Console.WriteLine(a);          // Point { X = 1, Y = 2 }

var (x, y) = a;               // deconstruction
```

### 6. Default Values

A struct's default value has **all fields zeroed** — numeric fields are 0, booleans are false, references are null. You get this with `default`:

```csharp
Point p = default;
Console.WriteLine(p.X); // 0
Console.WriteLine(p.Y); // 0
```

Since C# 10, you can define a custom parameterless constructor, but `default` still zeroes everything — it **bypasses your constructor**:

```csharp
struct Counter
{
    public int Value;
    public Counter() { Value = 1; }  // C# 10+
}

Counter a = new Counter();  // Value = 1 (constructor runs)
Counter b = default;         // Value = 0 (constructor does NOT run)
```

### 7. Passing Structs to Methods

Passing a struct by value copies it. For large structs this is expensive. Use `in` to pass by read-only reference:

```csharp
void Process(in LargeStruct data)
{
    // data is passed by reference — no copy
    // data is read-only — cannot be modified
}
```

See also: [[Method Parameter Modifiers]]

## When to Use Struct

Use a struct when **all** of these are true:

- **Small** — Microsoft recommends under 16 bytes
- **Immutable** or logically represents a single value (coordinates, color, date)
- **Short-lived** or created in high volume (game loops, math calculations, parsing)
- **Does not need inheritance**

Otherwise, use a class. Most types should be classes — struct is the performance optimization.

## Common Structs in .NET

These are all structs by design because they are small, frequently created, and represent single values:

- `int`, `double`, `bool`, `char` — all primitives
- `DateTime`, `TimeSpan`, `DateOnly`, `TimeOnly`
- `Guid`
- `Nullable<T>` (e.g. `int?`)
- `ValueTuple` (e.g. `(int, string)`)
- `Span<T>`, `ReadOnlySpan<T>` (ref struct)
