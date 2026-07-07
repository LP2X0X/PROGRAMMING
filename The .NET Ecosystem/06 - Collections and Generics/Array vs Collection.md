---
tags: 
 - csharp
 - collections
 - generics
---

## Overview

There are two main ways to store groups of data in C#: **arrays** and **collections**. The general term that covers both is simply **"collections"** (used loosely) or **"data structures"** (the computer science term).

> [!note]
> "Collection" is used in two ways in C#:
> - **Broad / informal** — any group of data, arrays included
> - **Specific / namespace** — the classes in `System.Collections` and `System.Collections.Generic`

```
Storing groups of data in C#
├── Arrays (int[], string[])
│     └── Fixed size, type-safe, fastest
│
└── Collections
      ├── Non-generic (System.Collections)
      │     └── ArrayList, Hashtable, Queue, Stack
      │         → stores object, boxing, legacy
      │
      └── Generic (System.Collections.Generic)  ✓ use this
            └── List<T>, Dictionary<K,V>, Queue<T>, Stack<T>, HashSet<T>
                → type-safe, no boxing, dynamic size
```

---

## Arrays

- Built into the language and the CLR itself — not a library class
- **Fixed size** — decided at creation, cannot grow or shrink
- **Type-safe** and **no boxing** — values stored directly
- Fastest option for indexed access

```csharp
int[] nums = new int[3];        // single dimension
int[,] grid = new int[3, 3];    // multi-dimensional
int[][] jagged = new int[3][];  // array of arrays
```

---

## Collections

Classes from the .NET library. **Dynamic size** — grow and shrink as needed. Come in two flavors:

| | Namespace | Type-safe | Boxing | Status |
|---|---|---|---|---|
| **Non-generic** | `System.Collections` | No | Yes | Legacy / obsolete |
| **Generic** | `System.Collections.Generic` | Yes | No | Modern standard |

---

## Comparison

| Feature                | Array (`int[]`)   | Non-generic (`ArrayList`)  | Generic (`List<int>`) |
| ---------------------- | ----------------- | -------------------------- | --------------------- |
| Type-safe              | Yes               | No — stores `object`       | Yes                   |
| Boxing for value types | No                | Yes                        | No                    |
| Size                   | Fixed at creation | Dynamic                    | Dynamic               |
| Performance            | Fastest           | Slowest (boxing + casting) | Fast                  |

---

## When to Use What

- **Array** — size is known and fixed, need raw speed for indexed access
- **Generic collection** (`List<T>`, `Dictionary<TKey, TValue>`, etc.) — need dynamic sizing, richer API, or a specific data structure
- **Non-generic collection** — avoid in new code, only encountered in legacy codebases
