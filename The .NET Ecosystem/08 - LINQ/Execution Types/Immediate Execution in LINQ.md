---
tags:
 - csharp
 - linq
---

Immediate execution is the opposite of [[Deferred Execution in LINQ|deferred execution]]. Instead of waiting until enumeration, the query runs **at the moment you call the method** and returns a concrete result — either a collection snapshot or a single value.

### Materializing into a Collection

When you need the results of a LINQ query outside of a `foreach` loop, you can force immediate execution by calling one of the materialization methods on `Enumerable`:

| Method | Returns |
|---|---|
| `ToArray<T>()` | `T[]` |
| `ToList<T>()` | `List<T>` |
| `ToDictionary<TSource, TKey>()` | `Dictionary<TKey, TSource>` |
| `ToHashSet<T>()` | `HashSet<T>` |

These methods enumerate the entire query **once** and store the results in a new collection. The returned collection is a **snapshot** — it is independent of the original data source, so later changes to the source will not be reflected.

```csharp
int[] numbers = { 10, 20, 30, 40, 1, 2, 3, 8 };

// Immediate execution: query runs right now and results are stored in a List<int>.
List<int> smallNumbers = numbers.Where(n => n < 10).ToList();

// Changing the source has no effect on the snapshot.
numbers[0] = 4;
// smallNumbers is still { 1, 2, 3, 8 }
```

### Returning a Single Element

Methods that return a single value also trigger immediate execution because they must inspect the sequence to produce their result:

| Method | Behavior |
|---|---|
| `First()` | Returns the first element. Throws if the sequence is empty. |
| `FirstOrDefault()` | Returns the first element, or the type's default value if the sequence is empty. |
| `Single()` | Returns the only element. Throws if the sequence is empty **or** contains more than one element. |
| `SingleOrDefault()` | Returns the only element, or the default value if the sequence is empty. Throws if more than one element exists. |

> [!tip]
> `First()` / `FirstOrDefault()` should typically be paired with `OrderBy()` or `OrderByDescending()` so the result is deterministic.

See also: [[Single vs First]] for a detailed comparison.

### Aggregate Methods

Aggregation methods also execute immediately, since they must walk the entire sequence to compute a result:

- `Count()`, `LongCount()`
- `Sum()`, `Average()`, `Min()`, `Max()`
- `Any()`, `All()`, `Contains()`
- `Aggregate()`

### Deferred vs. Immediate — When to Choose

| Use deferred execution when… | Use immediate execution when… |
|---|---|
| You want fresh results each time you enumerate | You need a stable snapshot that won't change |
| You are chaining multiple operators and want to avoid intermediate allocations | You need to pass results to code that expects a concrete collection (`List<T>`, `T[]`) |
| The data source may be large and you only need a subset | You want to iterate the results multiple times without re-querying |
