---
tags:
 - csharp
 - collections
 - generics
---

## What Is a Collection?

A **collection** is an object that groups multiple elements together and provides ways to store, retrieve, and manipulate them. Collections are part of the .NET class library and live primarily in `System.Collections` (non-generic) and `System.Collections.Generic` (generic/type-safe).

Think of a collection as a **smarter, more flexible container** than an array.

---

## Collections vs Arrays

| Feature | Array | Collection |
|---|---|---|
| **Size** | Fixed at creation (`new int[5]`) | Grows and shrinks dynamically |
| **Type safety** | Strongly typed (`int[]`, `string[]`) | Non-generic collections store `object` (boxing/unboxing); generic collections are type-safe |
| **Built-in methods** | Minimal — `Array.Sort()`, `Array.IndexOf()` | Rich API — `Add`, `Remove`, `Contains`, `Find`, `Sort`, etc. |
| **Performance** | Fastest for indexed access (`O(1)`) | Varies by type (list `O(1)` index, dictionary `O(1)` lookup, linked list `O(n)` index) |
| **Memory** | Contiguous block, no overhead | Extra overhead for internal bookkeeping (pointers, hash buckets, etc.) |
| **Flexibility** | Single dimension or multi-dimensional, one type | Many data structures: lists, queues, stacks, dictionaries, sets, etc. |

**When to use an array:** you know the exact size ahead of time and need raw speed for indexed access.
**When to use a collection:** you need dynamic sizing, frequent insertions/removals, or a richer data structure (key-value pairs, FIFO queue, etc.).

---

## The `System.Collections` Namespace (Non-Generic)

These were the original collection types shipped with .NET 1.0. They store everything as `object`, which means:
- **Boxing** when storing value types (`int` -> `object`) — performance cost.
- **No compile-time type safety** — you can accidentally mix types, and errors only surface at runtime via `InvalidCastException`.

### Key Types

| Type | Description |
|---|---|
| `ArrayList` | Dynamically sized list of `object`. Like `List<object>`. |
| `Hashtable` | Key-value pairs, both stored as `object`. Like `Dictionary<object, object>`. |
| `Queue` | FIFO (first-in, first-out) collection. |
| `Stack` | LIFO (last-in, first-out) collection. |
| `SortedList` | Key-value pairs sorted by key. |

### Example — ArrayList vs Array

```csharp
// Array — fixed size, type-safe
int[] nums = new int[3];
nums[0] = 10;
// nums[3] = 40;  // IndexOutOfRangeException

// ArrayList — dynamic size, stores object
ArrayList list = new ArrayList();
list.Add(10);        // boxing: int -> object
list.Add("hello");   // no compiler error, but mixing types is dangerous
list.Add(true);      // anything goes

int val = (int)list[0];  // must cast back — runtime error if wrong type
```

### The Boxing Problem

```csharp
ArrayList numbers = new ArrayList();
numbers.Add(42);    // int 42 is boxed into a heap-allocated object
int x = (int)numbers[0];  // unboxed back to int

// Each box/unbox is a small perf hit.
// With thousands of items, it adds up.
```

---

## Why Non-Generic Collections Are Mostly Obsolete

With .NET 2.0, **generic collections** (`System.Collections.Generic`) solved both problems:

| Non-Generic | Generic Replacement |
|---|---|
| `ArrayList` | `List<T>` |
| `Hashtable` | `Dictionary<TKey, TValue>` |
| `Queue` | `Queue<T>` |
| `Stack` | `Stack<T>` |
| `SortedList` | `SortedList<TKey, TValue>` |

> [!tip]
> In modern C# you should almost always use the generic versions. The non-generic types exist mainly for legacy code and for understanding .NET history.

---

## Key Interfaces in `System.Collections`

All non-generic collections implement one or more of these interfaces:

- **`ICollection`** — defines `Count`, `CopyTo`, `IsSynchronized`. Base for all collections.
- **`IEnumerable`** — provides `GetEnumerator()` so you can use `foreach`.
- **`IList`** — adds indexed access (`this[int index]`), `Add`, `Remove`, `Insert`. Implemented by `ArrayList`.
- **`IDictionary`** — adds key-value access (`this[object key]`), `Keys`, `Values`. Implemented by `Hashtable`.

```
IEnumerable
   └── ICollection
          ├── IList        (ArrayList, Array)
          └── IDictionary  (Hashtable, SortedList)
```

---

![[Pasted image 20240610170611.png|center|700]]
![[Pasted image 20240610170650.png|center|700]]