---
tags:
 - csharp
 - delegates
---

# Func Delegate

A built-in generic delegate that points to a method that **returns a value**. The **last** type parameter is always the return type. Exists in overloads from 0 to 16 input parameters:

```csharp
Func<TResult>                  // TResult ()         — no params
Func<T, TResult>               // TResult (T)        — one param
Func<T1, T2, TResult>          // TResult (T1, T2)   — two params
// ... up to Func<T1, ..., T16, TResult>
```

- Input parameters are **contravariant** (`in`) — you can assign `Func<object, int>` to `Func<string, int>`.
- Return type is **covariant** (`out`) — you can assign `Func<string>` to `Func<object>`.


---

## Basic Usage

```csharp
// No params, returns a value
Func<int> roll = () => new Random().Next(1, 7);
Console.WriteLine(roll()); // e.g. 4

// One param
Func<int, int> square = x => x * x;
Console.WriteLine(square(5)); // 25

// Two params
Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(3, 4)); // 7

// Multiple params, different types
Func<string, int, string> repeat = (text, count) => string.Concat(Enumerable.Repeat(text, count));
Console.WriteLine(repeat("Ha", 3)); // HaHaHa
```


---

## Reading the Type Signature

The last type is always the return. Everything before it is input:

```
Func<string, int, bool>
       ↑      ↑    ↑
     param1  param2  return type

// Equivalent to: bool Method(string arg1, int arg2)
```


---

## Passing Func as a Method Parameter

The main use case — injecting logic into a method:

```csharp
static List<T> Filter<T>(List<T> items, Func<T, bool> predicate)
{
    var result = new List<T>();
    foreach (var item in items)
    {
        if (predicate(item))
            result.Add(item);
    }
    return result;
}

var numbers = new List<int> { 1, 2, 3, 4, 5, 6 };

var evens = Filter(numbers, n => n % 2 == 0);   // [2, 4, 6]
var big   = Filter(numbers, n => n > 3);         // [4, 5, 6]
```

Different behavior, same method — the caller decides the logic.


---

## Pointing to Named Methods

```csharp
static bool IsPositive(int x) => x > 0;

Func<int, bool> check = IsPositive;
Console.WriteLine(check(-5)); // False
```


---

## Func in LINQ

Most LINQ methods take `Func` parameters. That's why you can pass lambdas to them:

```csharp
var names = new List<string> { "Alice", "Bob", "Charlie", "Dave" };

// Where expects Func<string, bool>
var longNames = names.Where(n => n.Length > 3).ToList();
// ["Alice", "Charlie", "Dave"]

// Select expects Func<string, TResult>
var upper = names.Select(n => n.ToUpper()).ToList();
// ["ALICE", "BOB", "CHARLIE", "DAVE"]

// OrderBy expects Func<string, TKey>
var sorted = names.OrderBy(n => n.Length).ToList();
// ["Bob", "Dave", "Alice", "Charlie"]
```


---

## Returning Func from a Method (Factory Pattern)

```csharp
static Func<int, int> GetMultiplier(int factor)
{
    return x => x * factor; // captures 'factor' via closure
}

var double_ = GetMultiplier(2);
var triple  = GetMultiplier(3);

Console.WriteLine(double_(5)); // 10
Console.WriteLine(triple(5));  // 15
```

The lambda captures the `factor` variable — this is a **closure**.


---

## Func vs Action vs Predicate

| Delegate | Returns | Use case |
|---|---|---|
| `Func<..., TResult>` | a value | transform, compute, query |
| `Action<...>` | `void` | side effects (log, write, modify) |
| `Predicate<T>` | `bool` | test a condition (shorthand for `Func<T, bool>`) |

```csharp
Func<int, int>    square    = x => x * x;          // returns int
Action<int>       print     = x => Console.WriteLine(x); // void
Predicate<int>    isEven    = x => x % 2 == 0;     // returns bool

// Predicate<T> is literally just Func<T, bool> with a different name.
// LINQ uses Func<T, bool>, List<T> methods use Predicate<T>.
```
