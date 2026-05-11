---
tags:
 - csharp
 - interfaces
---

The `IComparer<T>` interface defines custom comparison logic for sorting and ordering objects **externally** — meaning the comparison lives in a separate class, not inside the type being compared. This is the key distinction from `IComparable<T>`, which embeds a single "natural" ordering inside the type itself.

Use `IComparer<T>` when:
- You need **multiple sort orders** for the same type (by name, by age, by date, etc.).
- You don't own the type and **can't modify it** to add `IComparable<T>`.
- You want to keep comparison logic **decoupled** from domain classes.

---

## The Interface Definitions

### Non-Generic `IComparer` (System.Collections)

```csharp
public interface IComparer
{
    int Compare(object? x, object? y);
}
```

This is the legacy version from .NET 1.0. It works with `object`, so it requires casting and has no compile-time type safety. You'll encounter it with older APIs like `ArrayList.Sort()`.

### Generic `IComparer<T>` (System.Collections.Generic)

```csharp
public interface IComparer<in T>
{
    int Compare(T? x, T? y);
}
```

The generic version is strongly typed and **contravariant** (the `in` keyword). Contravariance means an `IComparer<object>` can be used where an `IComparer<string>` is expected, because if you can compare any two objects, you can certainly compare two strings.

### Return Value Contract

The `Compare(x, y)` method returns an `int` following this convention:

| Return Value   | Meaning               |
|----------------|-----------------------|
| Negative (`< 0`) | `x` precedes `y`     |
| Zero (`0`)       | `x` and `y` are equal |
| Positive (`> 0`) | `x` follows `y`       |

This is the same contract used by `IComparable<T>.CompareTo`, `string.Compare`, and the `Comparison<T>` delegate.

---

## The `Comparer<T>` Abstract Base Class

Rather than implementing `IComparer<T>` directly, you can inherit from `Comparer<T>` (in `System.Collections.Generic`). This abstract class:

- Implements both `IComparer<T>` and non-generic `IComparer` for you, so your comparer works with both old and new APIs.
- Provides `Comparer<T>.Default`, which returns a comparer that uses the type's `IComparable<T>` implementation.
- Provides the `Comparer<T>.Create()` factory method for quick inline comparers.

```csharp
public class AgeComparer : Comparer<Person>
{
    public override int Compare(Person? x, Person? y)
    {
        if (x is null && y is null) return 0;
        if (x is null) return -1;
        if (y is null) return 1;

        return x.Age.CompareTo(y.Age);
    }
}
```

By inheriting `Comparer<T>`, the non-generic `IComparer.Compare(object, object)` is automatically wired to call your typed override, handling the casting and type-check for you.

---

## Full Example: Multiple Sort Strategies

### The Domain Class

```csharp
public class Employee
{
    public string Name { get; set; }
    public int Age { get; set; }
    public decimal Salary { get; set; }
    public DateTime HireDate { get; set; }

    public override string ToString()
        => $"{Name} (Age: {Age}, Salary: {Salary:C}, Hired: {HireDate:d})";
}
```

### Comparer Implementations

```csharp
public class EmployeeAgeComparer : IComparer<Employee>
{
    public int Compare(Employee? x, Employee? y)
    {
        if (ReferenceEquals(x, y)) return 0;
        if (x is null) return -1;
        if (y is null) return 1;

        return x.Age.CompareTo(y.Age);
    }
}

public class EmployeeNameComparer : IComparer<Employee>
{
    public int Compare(Employee? x, Employee? y)
    {
        if (ReferenceEquals(x, y)) return 0;
        if (x is null) return -1;
        if (y is null) return 1;

        return string.Compare(x.Name, y.Name, StringComparison.OrdinalIgnoreCase);
    }
}

public class EmployeeSalaryDescendingComparer : IComparer<Employee>
{
    public int Compare(Employee? x, Employee? y)
    {
        if (ReferenceEquals(x, y)) return 0;
        if (x is null) return -1;
        if (y is null) return 1;

        // Swap x and y to get descending order
        return y.Salary.CompareTo(x.Salary);
    }
}
```

### Compound Comparer (Sort by Multiple Fields)

A compound comparer chains multiple comparisons — if the first field is equal, fall through to the next:

```csharp
public class EmployeeAgeThenNameComparer : IComparer<Employee>
{
    public int Compare(Employee? x, Employee? y)
    {
        if (ReferenceEquals(x, y)) return 0;
        if (x is null) return -1;
        if (y is null) return 1;

        int result = x.Age.CompareTo(y.Age);
        if (result != 0) return result;

        return string.Compare(x.Name, y.Name, StringComparison.OrdinalIgnoreCase);
    }
}
```

### Using the Comparers

```csharp
var employees = new List<Employee>
{
    new() { Name = "Alice",   Age = 30, Salary = 85000m, HireDate = new(2018, 3, 15) },
    new() { Name = "Bob",     Age = 25, Salary = 72000m, HireDate = new(2020, 7, 1) },
    new() { Name = "Charlie", Age = 30, Salary = 90000m, HireDate = new(2017, 1, 10) },
    new() { Name = "Diana",   Age = 28, Salary = 72000m, HireDate = new(2021, 11, 5) },
};

employees.Sort(new EmployeeAgeComparer());
// Bob(25), Diana(28), Alice(30), Charlie(30)

employees.Sort(new EmployeeSalaryDescendingComparer());
// Charlie(90k), Alice(85k), Bob(72k), Diana(72k)

employees.Sort(new EmployeeAgeThenNameComparer());
// Bob(25), Diana(28), Alice(30), Charlie(30)
```

---

## `Comparer<T>.Create()` — Quick Inline Comparers

When you don't want to write a whole class, `Comparer<T>.Create()` builds one from a `Comparison<T>` delegate:

```csharp
var byHireDate = Comparer<Employee>.Create(
    (x, y) => x.HireDate.CompareTo(y.HireDate)
);

employees.Sort(byHireDate);
```

This is convenient for one-off sorts but less reusable than a named class.

---

## The `Comparison<T>` Delegate

`List<T>.Sort()` also accepts a `Comparison<T>` delegate directly, which is just a method signature matching `int (T x, T y)`:

```csharp
employees.Sort((x, y) => x.Age.CompareTo(y.Age));

// Descending by salary using a lambda
employees.Sort((x, y) => y.Salary.CompareTo(x.Salary));
```

This is the most concise option for simple sorts. The difference from `IComparer<T>`:
- `IComparer<T>` is an **interface** — can be stored, injected, passed to constructors (like `SortedSet<T>`).
- `Comparison<T>` is a **delegate** — lighter syntax but only works with methods that accept it (like `List<T>.Sort`).

You can convert between them:

```csharp
// Delegate -> IComparer<T>
Comparison<Employee> comparison = (x, y) => x.Age.CompareTo(y.Age);
IComparer<Employee> comparer = Comparer<Employee>.Create(comparison);

// IComparer<T> -> Delegate (just call Compare)
IComparer<Employee> comp = new EmployeeAgeComparer();
employees.Sort(comp.Compare);  // method group conversion to Comparison<T>
```

---

## Where `IComparer<T>` Is Used in the BCL

Many framework types and methods accept an `IComparer<T>`:

| API | Purpose |
|-----|---------|
| `List<T>.Sort(IComparer<T>)` | Sort a list in-place |
| `Array.Sort<T>(T[], IComparer<T>)` | Sort an array in-place |
| `SortedSet<T>(IComparer<T>)` | Maintain a set in custom order |
| `SortedDictionary<K,V>(IComparer<K>)` | Dictionary sorted by keys |
| `SortedList<K,V>(IComparer<K>)` | Sorted list of key-value pairs |
| `Enumerable.OrderBy(..., IComparer<T>)` | LINQ ordering |
| `Array.BinarySearch<T>(T[], T, IComparer<T>)` | Binary search with custom comparison |
| `PriorityQueue<T, TPriority>(IComparer<TPriority>)` | Custom priority ordering (.NET 6+) |

### Example: SortedSet with Custom Order

Collections like `SortedSet<T>` accept a comparer at construction and use it for **all** insertions, lookups, and ordering:

```csharp
var byAge = new SortedSet<Employee>(new EmployeeAgeComparer());
byAge.Add(new Employee { Name = "Alice", Age = 30 });
byAge.Add(new Employee { Name = "Bob", Age = 25 });
// Iterating yields: Bob(25), Alice(30)
```

### Example: LINQ OrderBy with Comparer

```csharp
var sorted = employees
    .OrderBy(e => e.Name, StringComparer.OrdinalIgnoreCase)
    .ToList();
```

---

## Built-In Comparers

The framework ships several ready-made comparers:

### `StringComparer`

Provides static instances for common string comparison modes:

```csharp
StringComparer.Ordinal                // byte-by-byte, case-sensitive
StringComparer.OrdinalIgnoreCase      // byte-by-byte, case-insensitive
StringComparer.CurrentCulture         // culture-aware, case-sensitive
StringComparer.CurrentCultureIgnoreCase
StringComparer.InvariantCulture
StringComparer.InvariantCultureIgnoreCase
```

`StringComparer` implements both `IComparer<string>` and `IEqualityComparer<string>`, so it works with sorted collections and dictionaries alike:

```csharp
var caseInsensitiveSet = new SortedSet<string>(StringComparer.OrdinalIgnoreCase);
var caseInsensitiveDict = new Dictionary<string, int>(StringComparer.OrdinalIgnoreCase);
```

### `Comparer<T>.Default`

Returns a comparer that uses the type's own `IComparable<T>` implementation:

```csharp
Comparer<int>.Default.Compare(3, 5);    // -1 (negative)
Comparer<string>.Default.Compare("a", "b"); // negative
```

---

## Reversing a Comparer

There's no built-in `ReverseComparer<T>` in the BCL, but it's trivial to write a generic one:

```csharp
public class ReverseComparer<T> : IComparer<T>
{
    private readonly IComparer<T> _inner;

    public ReverseComparer(IComparer<T> inner) => _inner = inner;

    public int Compare(T? x, T? y) => _inner.Compare(y, x);
}

// Usage
var descendingAge = new ReverseComparer<Employee>(new EmployeeAgeComparer());
employees.Sort(descendingAge);
```

Or inline with `Comparer<T>.Create()`:

```csharp
var descending = Comparer<Employee>.Create(
    (x, y) => new EmployeeAgeComparer().Compare(y, x)
);
```

---

## Null Handling Conventions

The standard convention (followed by `Comparer<T>.Default` and all BCL comparers) is:

1. `null` is considered **less than** any non-null value.
2. Two `null` values are equal.
3. Use `ReferenceEquals(x, y)` as a fast path — if both reference the same object, return `0` immediately.

```csharp
public int Compare(T? x, T? y)
{
    if (ReferenceEquals(x, y)) return 0;  // same ref or both null
    if (x is null) return -1;
    if (y is null) return 1;

    // actual comparison logic
}
```

---

## `IComparer<T>` vs `IComparable<T>` — When to Use Which

| | `IComparable<T>` | `IComparer<T>` |
|---|---|---|
| **Where it lives** | Inside the type (`this.CompareTo(other)`) | External class |
| **How many** | One per type (the "natural" order) | Unlimited (one per strategy) |
| **Who implements** | The type being compared | A separate comparer class |
| **When to use** | There's one obvious default ordering (e.g., numbers, dates) | Multiple orderings, or you can't modify the type |
| **Works with** | Default sorting (`List.Sort()`, `SortedSet<T>()`) | Any API that accepts `IComparer<T>` |

A type can implement `IComparable<T>` for its default sort and still have multiple `IComparer<T>` implementations for alternative orderings. They complement each other.

---

## Summary

- **`IComparer<T>`** defines external, swappable comparison logic via `int Compare(T? x, T? y)`.
- Inherit from **`Comparer<T>`** instead of implementing `IComparer<T>` directly to get the non-generic interface for free.
- Use **`Comparer<T>.Create()`** or a **`Comparison<T>` lambda** for quick one-off sorts.
- The BCL uses `IComparer<T>` extensively — `List.Sort`, `Array.Sort`, `SortedSet<T>`, `SortedDictionary`, `PriorityQueue`, LINQ's `OrderBy`, and `BinarySearch` all accept one.
- Use **`StringComparer`** for string comparisons instead of writing your own.
- Follow the **null convention**: `null < non-null`, two nulls are equal, check `ReferenceEquals` first.
