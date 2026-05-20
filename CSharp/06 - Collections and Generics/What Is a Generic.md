---
tags:
 - csharp
 - generics
 - term
---

## Definition

A **generic** is a language feature that lets you define classes, methods, interfaces, and delegates with a **type parameter** — a placeholder for a type that gets filled in later by the consumer. Instead of writing code that operates on `object` (and losing type safety), you write it once with a placeholder like `T`, and the compiler generates type-safe, specialized versions for each concrete type you use.

```csharp
// T is the type parameter — a placeholder
public class Box<T>
{
    public T Value { get; set; }
}

// The consumer fills in T with a concrete type
Box<int> intBox = new Box<int>();       // T = int
intBox.Value = 42;                      // only accepts int

Box<string> strBox = new Box<string>(); // T = string
strBox.Value = "hello";                 // only accepts string
```

Without generics, you'd either write separate `BoxInt`, `BoxString`, etc. classes (code duplication), or use `object` and lose type safety + pay boxing costs.

---

## How the CLR Handles Generics

Generics in .NET are not just a compile-time trick (unlike Java's type erasure). The CLR has **native support** for generics — the type parameter information is preserved at runtime:

- **Value types** (`int`, `double`, `struct`, etc.) — the CLR generates a **separate specialized version** of the code for each value type. `List<int>` and `List<double>` are different types internally, each with their own optimized machine code. This is what eliminates boxing.
- **Reference types** (`string`, `Person`, etc.) — the CLR **shares one version** for all reference types, since they're all pointers of the same size under the hood.

```
List<int>    → CLR generates specialized int version    (no boxing)
List<double> → CLR generates specialized double version (no boxing)
List<string> ┐
List<Person> ┤→ CLR shares one reference-type version   (all are refs)
List<Car>    ┘
```

---

## Where You Can Use Generics

Generics aren't limited to classes. You can apply type parameters to:

### Generic Class

```csharp
public class Repository<T>
{
    private List<T> _items = new List<T>();
    public void Add(T item) => _items.Add(item);
    public T GetById(int index) => _items[index];
}
```

### Generic Method

A method can have its own type parameter, even inside a non-generic class:

```csharp
public class Utilities
{
    public static void Swap<T>(ref T a, ref T b)
    {
        T temp = a;
        a = b;
        b = temp;
    }
}

// Usage — compiler infers T from the arguments
int x = 1, y = 2;
Utilities.Swap(ref x, ref y);  // T inferred as int
```

### Generic Interface

```csharp
public interface IRepository<T>
{
    void Add(T item);
    T GetById(int id);
}
```

### Generic Delegate

```csharp
public delegate TResult Converter<TInput, TResult>(TInput input);

// Usage
Converter<string, int> toInt = s => int.Parse(s);
int result = toInt("42");  // result = 42
```

> [!tip]
> The built-in delegates `Func<T, TResult>`, `Action<T>`, and `Predicate<T>` are all generic delegates. You rarely need to define your own.

---

## Type Parameter Naming Conventions

| Convention | Used When | Example |
|---|---|---|
| `T` | Single type parameter | `List<T>`, `Box<T>` |
| `TKey`, `TValue` | Key-value relationships | `Dictionary<TKey, TValue>` |
| `TInput`, `TOutput` | Transformation / conversion | `Converter<TInput, TOutput>` |
| `TSource`, `TResult` | LINQ-style operations | `Func<TSource, TResult>` |

Always prefix with `T`. This is a strong .NET convention — `T` immediately signals "this is a type parameter, not a concrete type."

---

## The Three Problems Generics Solve

| Problem                  | Without Generics                              | With Generics                                    |
| ------------------------ | --------------------------------------------- | ------------------------------------------------ |
| **Code duplication**     | Write `BoxInt`, `BoxString`, `BoxPerson`...   | Write `Box<T>` once                              |
| **Type safety**          | Store as `object`, wrong type = runtime crash | Compiler catches type mismatches at compile time |
| **Performance (boxing)** | Value types boxed into heap objects           | Value types stored directly, no boxing           |

---

## Quick Mental Model

Think of a generic like a **template with a blank**:

```
class Box<___>          ← definition: "a box that holds ___"

Box<int>                ← usage: "a box that holds int"
Box<string>             ← usage: "a box that holds string"
```

The blank (`T`) gets filled in when you use it. The compiler then enforces that everything inside the class is consistent with that choice — you can't accidentally put a `string` in a `Box<int>`.
