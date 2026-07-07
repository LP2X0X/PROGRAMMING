---
tags:
 - csharp
 - linq
---

Paging in LINQ means selecting a subset of a sequence — the first N items, the last N, a middle slice, or fixed-size batches. LINQ provides three approaches: **Take/Skip methods**, **Range syntax with Take**, and **Chunk**.

All paging operators are extension methods on `IEnumerable<T>` defined in `System.Linq.Enumerable`. They use [[Deferred Execution in LINQ|deferred execution]] — no elements are consumed until you enumerate the result.

---

## Take and Skip Family

These are the classic paging operators. They come in pairs — `Take` selects elements, `Skip` passes over them.

| Method | What it does | Added in |
|---|---|---|
| `Take(n)` | Returns the first `n` elements | .NET 3.5 |
| `TakeLast(n)` | Returns the last `n` elements | .NET Core 2.0 |
| `TakeWhile(predicate)` | Returns elements from the start as long as the condition is true — stops at the first `false` | .NET 3.5 |
| `Skip(n)` | Skips the first `n` elements, returns the rest | .NET 3.5 |
| `SkipLast(n)` | Skips the last `n` elements, returns the rest | .NET Core 2.0 |
| `SkipWhile(predicate)` | Skips elements from the start as long as the condition is true — returns everything after the first `false` | .NET 3.5 |

### Take(n) and Skip(n) — The Fundamentals

`Take(n)` returns **at most** `n` elements from the beginning of a sequence. If the sequence has fewer than `n` elements, it simply returns everything — no exception is thrown.

`Skip(n)` discards the first `n` elements and returns the rest. If you skip past the end, you get an empty sequence.

```csharp
int[] numbers = { 10, 20, 30, 40, 50 };

var firstThree = numbers.Take(3);   // { 10, 20, 30 }
var afterThree = numbers.Skip(3);   // { 40, 50 }

// Edge cases — no exceptions
var takeTooMany = numbers.Take(100); // { 10, 20, 30, 40, 50 } — returns all
var skipTooMany = numbers.Skip(100); // { } — empty sequence
var takeZero    = numbers.Take(0);   // { } — empty sequence
var skipZero    = numbers.Skip(0);   // { 10, 20, 30, 40, 50 } — returns all
```

### TakeLast(n) and SkipLast(n)

These mirror `Take`/`Skip` but operate from the **end** of the sequence.

```csharp
int[] numbers = { 10, 20, 30, 40, 50 };

var lastTwo    = numbers.TakeLast(2);  // { 40, 50 }
var dropLastTwo = numbers.SkipLast(2); // { 10, 20, 30 }
```

> [!note] Implementation detail
> `TakeLast` and `SkipLast` must buffer elements internally because `IEnumerable<T>` is forward-only — the sequence doesn't know its length upfront. They use a circular buffer of size `n` under the hood, so memory usage is proportional to `n`, not to the total sequence length.

### Basic Paging Pattern

Combine `Skip` and `Take` to implement page-based navigation:

```csharp
int pageSize = 10;
int pageNumber = 3; // zero-indexed: page 0, 1, 2, ...

var page = products
    .Skip(pageNumber * pageSize)
    .Take(pageSize);
```

A reusable helper:

```csharp
public static IEnumerable<T> Page<T>(this IEnumerable<T> source, int pageNumber, int pageSize)
    => source.Skip(pageNumber * pageSize).Take(pageSize);

// Usage
var thirdPage = products.Page(pageNumber: 2, pageSize: 10);
```

> [!warning] Performance on large in-memory collections
> `Skip(n)` still iterates through the first `n` elements — it just doesn't yield them. For page 1000 with page size 10, it walks through 10,000 elements to reach the ones you want. This is fine for moderate-sized collections but can be a problem at scale. For database queries via `IQueryable<T>`, the provider translates `Skip`/`Take` into SQL `OFFSET`/`FETCH` (or `LIMIT`/`OFFSET`), so the database handles the skipping efficiently.

### TakeWhile / SkipWhile

These are **not** the same as `Where`. They operate **sequentially from the start** and stop the moment the condition flips. Order matters.

```csharp
int[] numbers = { 2, 4, 6, 1, 8, 10 };

var taken = numbers.TakeWhile(n => n % 2 == 0);
// { 2, 4, 6 } — stops at 1 because 1 is odd, even though 8 and 10 come later

var skipped = numbers.SkipWhile(n => n % 2 == 0);
// { 1, 8, 10 } — skips 2, 4, 6, then returns everything from 1 onward
```

Compare this with `Where`:

```csharp
var filtered = numbers.Where(n => n % 2 == 0);
// { 2, 4, 6, 8, 10 } — examines every element, returns all matches regardless of position
```

The key difference:
- `Where` scans the **entire** sequence and returns **all** matches.
- `TakeWhile` scans from the start and **stops** at the first non-match.
- `SkipWhile` scans from the start and **starts yielding** at the first non-match.

#### Overload with Index

Both `TakeWhile` and `SkipWhile` have an overload that provides the element's index as a second parameter:

```csharp
string[] names = { "Alice", "Anna", "Bob", "Carol", "Cathy" };

// Take while the name starts with 'A' AND we haven't taken more than 3
var result = names.TakeWhile((name, index) => name.StartsWith("A") && index < 3);
// { "Alice", "Anna" }
```

#### When to Use TakeWhile / SkipWhile

They are most useful when data is **already sorted** in a meaningful order:

```csharp
// Sorted log entries — take all entries from today
var todaysLogs = logs
    .OrderByDescending(l => l.Timestamp)
    .TakeWhile(l => l.Timestamp.Date == DateTime.Today);

// Sorted scores — skip everyone who passed, get the failures
var failures = scores
    .OrderByDescending(s => s.Value)
    .SkipWhile(s => s.Value >= 60);
```

### Combining Take and Skip Operations

You can chain these operators to express more complex slicing:

```csharp
int[] numbers = { 10, 20, 30, 40, 50, 60, 70, 80 };

// Get elements 3 through 5 (zero-indexed: indices 2, 3, 4)
var slice = numbers.Skip(2).Take(3);  // { 30, 40, 50 }

// Get everything except the first 2 and last 2
var middle = numbers.Skip(2).SkipLast(2);  // { 30, 40, 50, 60 }

// Get the 2nd-to-last element only
var secondToLast = numbers.TakeLast(2).Take(1);  // { 70 }
```

---

## Range Syntax with Take (.NET 6 / C# 10)

Starting in .NET 6, `Take()` accepts an [[Indices and Ranges|Index/Range]] argument, which lets you express paging in a single call instead of combining `Skip` and `Take`.

### Range Notation Quick Reference

| Syntax | Meaning |
|---|---|
| `..n` | From the start up to (but not including) index `n` |
| `n..` | From index `n` to the end |
| `n..m` | From index `n` up to (but not including) index `m` |
| `^n..` | From `n` positions before the end to the end |
| `..^n` | From the start up to `n` positions before the end |
| `^n..^m` | From `n` before the end up to `m` before the end |

### Examples

```csharp
var products = GetProducts(); // assume 10 products, indices 0–9

// First 3 items (equivalent to Take(3))
var first3 = products.Take(..3);

// Skip first 3 (equivalent to Skip(3))
var skipFirst3 = products.Take(3..);

// Last 2 items (equivalent to TakeLast(2))
var last2 = products.Take(^2..);

// Skip 3, then take 2 (equivalent to Skip(3).Take(2))
var middle = products.Take(3..5);

// Everything except the last 2 (equivalent to SkipLast(2))
var skipLast2 = products.Take(..^2);

// Items from 3rd-to-last up to (but not including) the last
var nearEnd = products.Take(^3..^1);  // equivalent to TakeLast(3).SkipLast(1)
```

### Side-by-Side: Skip/Take vs Range Syntax

| Skip/Take | Range | Result (on 10 items) |
|---|---|---|
| `.Take(3)` | `.Take(..3)` | Items at index 0, 1, 2 |
| `.Skip(3)` | `.Take(3..)` | Items at index 3–9 |
| `.TakeLast(2)` | `.Take(^2..)` | Items at index 8, 9 |
| `.SkipLast(2)` | `.Take(..^2)` | Items at index 0–7 |
| `.Skip(3).Take(2)` | `.Take(3..5)` | Items at index 3, 4 |
| `.TakeLast(3).SkipLast(1)` | `.Take(^3..^1)` | Items at index 7, 8 |

The range syntax is more concise and makes the intent clearer, especially for slicing from the end with the `^` operator.

### Edge Cases

Just like `Take(n)`, ranges that go beyond the sequence length don't throw — they return as many elements as available:

```csharp
int[] numbers = { 1, 2, 3 };

var result = numbers.Take(..100);  // { 1, 2, 3 } — no exception
var empty  = numbers.Take(50..60); // { } — empty, start is past the end
```

---

## Chunk (.NET 6 / C# 10)

`Chunk(size)` splits a sequence into batches of a fixed size. The return type is `IEnumerable<T[]>` — each chunk is an array.

```csharp
var products = GetProducts(); // 7 products

var batches = products.Chunk(3);
// Batch 1: [product0, product1, product2]
// Batch 2: [product3, product4, product5]
// Batch 3: [product6]              — last chunk holds the remainder
```

### How It Works Internally

`Chunk` uses [[Immediate Execution in LINQ|immediate execution]] **per chunk** — each inner array is fully materialized when you advance to it, but the next chunk isn't built until you ask for it. The outer `IEnumerable<T[]>` is still deferred.

### Practical Use Cases

#### Batch database inserts

```csharp
var allRecords = GetRecordsToInsert(); // 10,000 records

foreach (var batch in allRecords.Chunk(500))
{
    // batch is T[] with at most 500 elements
    dbContext.BulkInsert(batch);
}
```

#### Display data in a grid (rows of N columns)

```csharp
var items = GetMenuItems();

foreach (var row in items.Chunk(4))
{
    // Each row has up to 4 items — render as a grid row
    Console.WriteLine(string.Join(" | ", row.Select(i => i.Name.PadRight(20))));
}
```

#### Parallel processing in batches

```csharp
var workItems = GetWorkItems();

foreach (var batch in workItems.Chunk(100))
{
    await Task.WhenAll(batch.Select(item => ProcessAsync(item)));
}
```

### Edge Cases

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

var chunks = numbers.Chunk(5);  // one chunk: [1, 2, 3, 4, 5]
var big    = numbers.Chunk(100); // one chunk: [1, 2, 3, 4, 5] — smaller than size, that's fine
var single = numbers.Chunk(1);  // five chunks: [1], [2], [3], [4], [5]

// Chunk(0) or Chunk(-1) throws ArgumentOutOfRangeException
```

### Before .NET 6: Manual Chunking

If you're targeting an older framework, you can achieve the same result with `Skip`/`Take`:

```csharp
public static IEnumerable<IEnumerable<T>> ChunkManual<T>(this IEnumerable<T> source, int size)
{
    var list = source.ToList();
    for (int i = 0; i < list.Count; i += size)
    {
        yield return list.Skip(i).Take(size);
    }
}
```

This materializes the entire sequence first (unlike the built-in `Chunk`), so prefer the native method when available.

---

## Comparison

| Approach | Best for | Available since |
|---|---|---|
| `Skip(n).Take(n)` | Classic page-number paging | .NET 3.5 |
| `TakeLast(n)` / `SkipLast(n)` | Slicing from the end | .NET Core 2.0 |
| `TakeWhile` / `SkipWhile` | Condition-based slicing on ordered data | .NET 3.5 |
| `Take(range)` | Concise slicing, especially from the end | .NET 6 |
| `Chunk(size)` | Fixed-size batches for batch processing | .NET 6 |

### Decision Guide

```
Do you need fixed-size batches?
  └─ Yes → Chunk(size)
  └─ No
      ├─ Do you know the exact positions (indices)?
      │    └─ Yes → Take(range) if .NET 6+, otherwise Skip(n).Take(n)
      ├─ Do you need elements based on a condition in sorted data?
      │    └─ Yes → TakeWhile / SkipWhile
      └─ Do you need elements from the end?
           └─ Yes → TakeLast(n) / SkipLast(n), or Take(^n..) if .NET 6+
```
