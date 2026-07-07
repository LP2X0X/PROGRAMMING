---
tags:
 - csharp
 - basics
 - methods
---

## What Is a Local Function?

A local function is a method defined **inside another method**. It's only accessible within the containing method — it doesn't pollute the class's API. Local functions can access variables and parameters from the enclosing method (this is called **capturing** or **closure**).

```csharp
public void Process(int[] numbers)
{
    int threshold = 10;

    // Local function — can access 'threshold' and 'numbers' from the enclosing method
    bool IsValid(int n)
    {
        return n > 0 && n < threshold;
    }

    foreach (var n in numbers)
    {
        if (IsValid(n))
            Console.WriteLine(n);
    }
}
```

---

## Placement: Before or After — It Doesn't Matter

Unlike regular method calls, you can **call a local function before its declaration** in the method body. The compiler handles it either way:

```csharp
public void Example()
{
    SayHello();  // works — even though SayHello is defined below

    void SayHello() => Console.WriteLine("Hello");
}
```

Convention is to place local functions at the **end** of the containing method, to keep the main logic up top.

---

## Static Local Functions

By default, local functions can capture variables from the enclosing method. Marking a local function `static` **prevents this** — it can only use its own parameters and static members:

```csharp
public void MyMethod()
{
    int x = 10;

    // Normal — captures x from enclosing method
    void Add(int y) => Console.WriteLine(x + y);

    // Static — isolated, cannot touch x
    static void AddStatic(int a, int b) => Console.WriteLine(a + b);

    // This would NOT compile:
    // static void Broken(int y) => Console.WriteLine(x + y);  // ERROR — can't capture x
}
```

### Why use `static`?

When a local function captures outer variables, the compiler generates a hidden class (a closure) to hold them — this has a small allocation cost. `static` prevents accidental captures and avoids that overhead. It's a way of saying "this function is self-contained, pass everything in as parameters."

---

## Local Functions Can Be Recursive

```csharp
public int Factorial(int n)
{
    return Compute(n);

    int Compute(int value)
    {
        if (value <= 1) return 1;
        return value * Compute(value - 1);  // calls itself
    }
}
```

---

## Local Functions Can Be Generic

```csharp
public void Example()
{
    Print(42);
    Print("hello");

    void Print<T>(T value) => Console.WriteLine(value);
}
```

---

## Local Functions Support Attributes

Since C# 9, you can apply attributes to local functions and their parameters:

```csharp
public void Example()
{
    [System.Diagnostics.Conditional("DEBUG")]
    static void DebugLog(string msg) => Console.WriteLine(msg);

    DebugLog("only runs in DEBUG builds");
}
```

---

## Local Function vs Lambda

Both let you define inline logic, but they have different trade-offs:

| | Local Function | Lambda |
|---|---|---|
| Syntax | `void Foo() { }` | `Action foo = () => { };` |
| Can be `static` | Yes — prevents captures | No |
| Can be generic | Yes — `void Foo<T>()` | No |
| Can be recursive | Yes — calls itself by name | Awkward — variable must be declared first |
| Allocation | None (unless captured as delegate) | Allocates a delegate object |
| Supports `ref`/`out`/`in` params | Yes | No |
| Supports `yield return` | Yes | No |
| Supports `async`/`await` | Yes | Yes |
| Can use attributes | Yes (C# 9+) | No |

> [!tip]
> Prefer local functions over lambdas when the logic is only used within the method. They're more capable (generics, recursion, `ref` params, `yield`) and avoid delegate allocation.

---

## Iterator and Async Validation Pattern

A common use case for local functions is **eager argument validation** in iterators and async methods. Without a local function, exceptions are deferred:

```csharp
// Problem: ArgumentException is NOT thrown until someone iterates the result
public IEnumerable<int> GetNumbers(int count)
{
    if (count < 0) throw new ArgumentException("count must be >= 0");
    for (int i = 0; i < count; i++)
        yield return i;
}

// Solution: local function separates validation (eager) from iteration (lazy)
public IEnumerable<int> GetNumbers(int count)
{
    if (count < 0) throw new ArgumentException("count must be >= 0");  // throws immediately
    return Generate();

    IEnumerable<int> Generate()
    {
        for (int i = 0; i < count; i++)
            yield return i;
    }
}
```

The same pattern applies to `async` methods where you want to validate parameters before entering the async state machine.

---

## Summary

- Local functions are methods inside methods, scoped to the containing method only
- They can capture enclosing variables (closure) unless marked `static`
- Use `static` to prevent accidental captures and avoid closure allocation
- They support generics, recursion, `ref`/`out`/`in`, `yield return`, and attributes
- Prefer them over lambdas for method-internal helper logic
- Common pattern: eager validation + local function for deferred `yield`/`async` logic