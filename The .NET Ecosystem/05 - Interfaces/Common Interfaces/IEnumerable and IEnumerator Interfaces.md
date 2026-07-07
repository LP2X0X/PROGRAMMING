---
tags:
 - csharp
 - interfaces
---

## `IEnumerable<T>` and `IEnumerator<T>`

These two interfaces are the foundation of iteration in .NET. Every `foreach` loop, every LINQ query, and every collection-based API depends on them.

| | `IEnumerable<T>` | `IEnumerator<T>` |
|---|---|---|
| **Role** | "I am a collection that **can be** iterated" | "I am an **active iteration** in progress" |
| **Method** | `GetEnumerator()` | `MoveNext()`, `Current`, `Reset()` |
| **Analogy** | A book | A bookmark |

---

### `IEnumerable<T>` Interface

Defined in `System.Collections.Generic`. Has a single method that returns an enumerator:

```csharp
public interface IEnumerable<out T> : IEnumerable
{
    IEnumerator<T> GetEnumerator();
}
```

### `IEnumerator<T>` Interface

The cursor that tracks position and walks through the collection:

```csharp
public interface IEnumerator<out T> : IDisposable, IEnumerator
{
    T Current { get; }
}

// Inherited from IEnumerator:
public interface IEnumerator
{
    bool MoveNext();
    object Current { get; }
    void Reset();
}
```

- **`MoveNext()`** -- advances to the next element. Returns `false` when past the end.
- **`Current`** -- the element at the current position. Invalid before the first `MoveNext()` or after `MoveNext()` returns `false`.
- **`Reset()`** -- moves back to before the first element. Often throws `NotSupportedException` -- most enumerators don't support it.
- **`Dispose()`** -- cleans up resources (from `IDisposable`). Called automatically by `foreach`.

---

### Why Two Separate Interfaces?

This is the most important design question: why doesn't `IEnumerable` just have `MoveNext()` and `Current` directly?

**The core problem: multiple simultaneous iterations.**

If the collection itself tracked the current position, only one `foreach` could run at a time:

```csharp
// IMAGINE if the collection tracked position directly
class BrokenList<T>
{
    private int _position; // shared state -- disaster

    public T Current => _items[_position];
    public bool MoveNext() => ++_position < _items.Count;
}
```

Nested loops would break:

```csharp
var list = new List<string> { "A", "B", "C" };

foreach (var x in list)          // iteration 1
{
    foreach (var y in list)      // iteration 2 -- needs its own position!
    {
        Console.Write($"{x}{y} ");
    }
}
// Expected: AA AB AC BA BB BC CA CB CC
// With shared state: inner loop would corrupt outer loop's position
```

**The separation solves it.** Each call to `GetEnumerator()` returns a **new, independent** enumerator with its own position:

```csharp
foreach (var x in list)       // calls list.GetEnumerator() → enumerator #1
{
    foreach (var y in list)   // calls list.GetEnumerator() → enumerator #2
    {
        Console.Write($"{x}{y} ");
    }
}
// Works correctly: AA AB AC BA BB BC CA CB CC
```

Each enumerator independently tracks where it is:

```csharp
IEnumerator<int> e1 = list.GetEnumerator(); // position: before first
IEnumerator<int> e2 = list.GetEnumerator(); // position: before first

e1.MoveNext(); // e1 → index 0, e2 still before first
e1.MoveNext(); // e1 → index 1, e2 still before first
e2.MoveNext(); // e2 → index 0, e1 still at index 1
```

**It's the Factory Pattern**: `IEnumerable` is a factory that produces independent `IEnumerator` instances:

```
IEnumerable (the collection)
    │
    ├── GetEnumerator() → IEnumerator #1 (foreach loop 1)
    ├── GetEnumerator() → IEnumerator #2 (foreach loop 2)
    └── GetEnumerator() → IEnumerator #3 (LINQ query)
```

---

### What `foreach` Actually Does

```csharp
// This:
foreach (var item in collection)
{
    Console.WriteLine(item);
}

// Is compiled into this:
IEnumerator<T> e = collection.GetEnumerator();
try
{
    while (e.MoveNext())
    {
        T item = e.Current;
        Console.WriteLine(item);
    }
}
finally
{
    e.Dispose();
}
```

---

### Custom Collection Example

A full implementation showing both interfaces:

```csharp
public class CustomCollection<T> : IEnumerable<T>
{
    private readonly T[] _items;

    public CustomCollection(T[] items) => _items = items;

    public IEnumerator<T> GetEnumerator() => new CustomEnumerator(_items);

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();

    private class CustomEnumerator : IEnumerator<T>
    {
        private readonly T[] _items;
        private int _index = -1;

        public CustomEnumerator(T[] items) => _items = items;

        public T Current
        {
            get
            {
                if (_index < 0 || _index >= _items.Length)
                    throw new InvalidOperationException();
                return _items[_index];
            }
        }

        object IEnumerator.Current => Current;

        public bool MoveNext() => ++_index < _items.Length;

        public void Reset() => _index = -1;

        public void Dispose() { }
    }
}
```

Usage:

```csharp
var collection = new CustomCollection<int>([1, 2, 3, 4, 5]);

foreach (var item in collection)
{
    Console.WriteLine(item);
}
```

---

### Simplified with `yield return`

In practice, you rarely write a manual `IEnumerator<T>`. The `yield return` keyword generates one for you:

```csharp
public class CustomCollection<T> : IEnumerable<T>
{
    private readonly T[] _items;

    public CustomCollection(T[] items) => _items = items;

    public IEnumerator<T> GetEnumerator()
    {
        for (int i = 0; i < _items.Length; i++)
            yield return _items[i];
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

The compiler generates a hidden class that implements `IEnumerator<T>` with all the state tracking. Each call to `GetEnumerator()` still returns a new independent instance.

---

### Warning: `yield return` Methods Are Entirely Deferred

None of the code inside a `yield return` method runs when you call it. The entire body is postponed until the first `MoveNext()` -- meaning the first time you iterate. This includes validation, logging, and exceptions.

```csharp
public IEnumerable<int> GetNumbers(int[] data)
{
    Console.WriteLine("START");

    if (data == null)
        throw new ArgumentNullException(nameof(data));

    foreach (var item in data)
        yield return item;
}
```

```csharp
var result = GetNumbers(null);   // NOTHING HAPPENS. No print. No exception.

Console.WriteLine("Before loop");

foreach (var n in result)        // NOW it runs. NOW it throws.
{
    Console.WriteLine(n);
}

// Output:
// Before loop
// START
// Unhandled exception: ArgumentNullException
```

**Why?** The compiler transforms the entire method body into a state machine (a hidden `IEnumerator<T>` class with a `_state` field and a `switch` inside `MoveNext()`). Calling the method just creates that object -- no code from your method body executes until `MoveNext()` is called.

```
Your call:   GetNumbers(null)
What runs:   return new <GetNumbers>d__1 { _data = null, _state = 0 };
             // that's it -- constructor only, no body execution

First loop:  foreach (var n in result)  →  calls MoveNext()
What runs:   YOUR code finally executes  →  hits the null check  →  throws
```

This means validation errors surface far from the actual bug:

```csharp
var items = GetNumbers(null);     // line 10 -- bug is HERE

// ... 200 lines later ...

foreach (var item in items) { }   // line 210 -- exception shows up HERE
```

**The fix: split validation from iteration.**

```csharp
// Public method: no yield, validates IMMEDIATELY
public IEnumerable<int> GetNumbers(int[] data)
{
    if (data == null)
        throw new ArgumentNullException(nameof(data)); // throws at call site

    return GetNumbersIterator(data);
}

// Private iterator: deferred execution
private IEnumerable<int> GetNumbersIterator(int[] data)
{
    foreach (var item in data)
        yield return item;
}
```

```csharp
var result = GetNumbers(null);  // throws immediately -- good!
```

**Rule of thumb**: if a `yield return` method accepts parameters that could be invalid, wrap it in a non-iterator method that validates first.

---

### Tips & Best Practices

- **Implement both generic and non-generic versions.** `IEnumerable<T>` extends `IEnumerable`, so you must implement `IEnumerable.GetEnumerator()` too. Use explicit interface implementation and delegate to the generic version.
- **Use `yield return` over manual enumerators.** It's shorter, less error-prone, and the compiler handles `Dispose` and state management correctly.
- **Never modify a collection during enumeration.** Most .NET collections throw `InvalidOperationException` if you add/remove items while a `foreach` is active. Collect changes and apply them after the loop.
- **`IEnumerable<T>` enables LINQ.** All LINQ extension methods (`Where`, `Select`, `OrderBy`, etc.) are defined on `IEnumerable<T>`. Implementing it gives your collection the entire LINQ API for free.
- **Be aware of deferred execution.** `IEnumerable<T>` returned from `yield return` or LINQ is lazy -- it doesn't execute until you iterate. This is powerful but can surprise you if the underlying data changes between creation and iteration.
- **Don't call `Reset()`.** It's a legacy holdover. Many enumerators throw `NotSupportedException`. Create a new enumerator instead.
- **`foreach` calls `Dispose()` automatically.** This is why `IEnumerator<T>` extends `IDisposable` -- to clean up resources (like database connections or file handles) when iteration ends, even if it ends early via `break` or an exception.
