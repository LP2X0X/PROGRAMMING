---
tags:
 - csharp
 - basics
 - types
---

## What It Is

- Lives in `System.Text` namespace
- A **mutable** sequence of characters — the counterpart to immutable `string`
- Designed for scenarios where you build or modify strings repeatedly without allocating a new object each time
- A **class** (reference type), not a struct

```csharp
using System.Text;

var sb = new StringBuilder();
sb.Append("hello");
sb.Append(" ");
sb.Append("world");
string result = sb.ToString();  // "hello world"
```

---

## Why It Exists — The Problem It Solves

Strings are immutable. Every concatenation creates a **new string on the heap**:

```csharp
// BAD — O(n^2) time and memory
string s = "";
for (int i = 0; i < 10000; i++)
    s += i.ToString();
// Each += allocates a new string and copies all previous content
// Iteration 9999 copies ~40,000 characters just to append a few more
```

StringBuilder mutates an internal buffer in place:

```csharp
// GOOD — O(n) time and memory
var sb = new StringBuilder();
for (int i = 0; i < 10000; i++)
    sb.Append(i);
string s = sb.ToString();
```

---

## Internal Architecture

### Chunk-Based Linked List (not a single array)

Prior to .NET Core, StringBuilder used a single `char[]` that was resized (copied) when full. Modern .NET uses a **linked list of chunks**:

```
StringBuilder (current chunk)
  ├── m_ChunkChars: char[8000]    ← most recent writes go here
  ├── m_ChunkLength: 5432         ← how much of this chunk is used
  └── m_ChunkPrevious ──► StringBuilder (previous chunk)
                            ├── m_ChunkChars: char[8000]
                            ├── m_ChunkLength: 8000
                            └── m_ChunkPrevious ──► ...
```

**Key implications:**
- **Appending is O(1) amortized** — when the current chunk fills, a new chunk is allocated without copying old data
- **Indexing `sb[i]` is O(n) in chunks** — must walk the linked list to find the right chunk (but chunks are large, so in practice it's fast)
- **`ToString()` is O(n)** — copies all chunks into one contiguous string

### Default Capacity

```csharp
new StringBuilder()        // default capacity: 16 characters
new StringBuilder(1000)    // pre-allocate for 1000 characters
new StringBuilder("init")  // capacity = "init".Length + 16
```

If you know the approximate final size, set the capacity upfront to avoid chunk allocations:

```csharp
// Processing 10,000 items averaging ~6 chars each
var sb = new StringBuilder(10000 * 6);
```

---

## Core API

### Append Family

```csharp
var sb = new StringBuilder();

sb.Append("text");              // append string
sb.Append('c');                 // append char
sb.Append(42);                  // append int (no boxing — dedicated overload)
sb.Append(3.14);                // append double
sb.Append("hello", 0, 3);      // append substring: "hel"
sb.Append('=', 20);             // append char repeated: "===================="

sb.AppendLine("next line");     // append + Environment.NewLine
sb.AppendLine();                // just a newline

// AppendFormat — like String.Format but writes into the buffer
sb.AppendFormat("Name: {0}, Age: {1}", name, age);

// AppendJoin (.NET Core 2.0+) — like String.Join but into the buffer
sb.AppendJoin(", ", new[] { "a", "b", "c" });  // "a, b, c"
```

### Insert

```csharp
sb.Insert(0, "prefix: ");       // insert at index 0
sb.Insert(5, '*', 3);           // insert char repeated 3 times at index 5
```

> Insert is **expensive** — it must shift all content after the insertion point. Avoid in hot paths.

### Remove and Replace

```csharp
sb.Remove(0, 5);                         // remove 5 chars starting at index 0
sb.Replace("old", "new");                // replace all occurrences
sb.Replace("old", "new", 10, 20);       // replace within a range
sb.Replace('a', 'b');                    // replace all chars
```

### Clear

```csharp
sb.Clear();          // resets Length to 0, but keeps the allocated buffer
// Equivalent to sb.Length = 0;
// The capacity (allocated memory) is NOT released — good for reuse
```

### Indexer

```csharp
sb[0] = 'H';                    // mutate individual characters
char first = sb[0];             // read individual characters
```

### Properties

```csharp
sb.Length;        // number of characters currently in the builder
sb.Capacity;     // size of the allocated buffer
sb.MaxCapacity;  // hard upper limit (default: int.MaxValue)

// You can truncate by setting Length
sb.Length = 10;   // keeps only the first 10 characters
```

---

## Method Chaining (Fluent API)

Most methods return `this`, enabling chaining:

```csharp
string result = new StringBuilder()
    .Append("SELECT ")
    .Append(columns)
    .Append(" FROM ")
    .Append(table)
    .Append(" WHERE ")
    .Append(condition)
    .ToString();
```

---

## Span Integration (.NET Core 2.1+ / .NET 5+)

### Writing from Spans

```csharp
ReadOnlySpan<char> data = "hello world".AsSpan(0, 5);
sb.Append(data);  // no intermediate string allocation
```

### Reading Chunks Without ToString

```csharp
// Enumerate chunks without allocating a single string
foreach (ReadOnlyMemory<char> chunk in sb.GetChunks())
{
    ReadOnlySpan<char> span = chunk.Span;
    // Process span directly — zero allocation
}
```

### TryCopyTo

```csharp
Span<char> buffer = stackalloc char[sb.Length];
sb.CopyTo(0, buffer, sb.Length);
// Now you have the content on the stack — no heap allocation at all
```

---

## Interpolated String Handler (.NET 6+ / C# 10+)

StringBuilder has its own `AppendInterpolatedStringHandler`, so `Append` and `AppendLine` accept interpolated strings **without allocating a temporary string**:

```csharp
// This does NOT create an intermediate string
// The handler writes each piece directly into the StringBuilder's buffer
sb.Append($"Name: {name}, Age: {age}");

// Equivalent to:
sb.Append("Name: ");
sb.Append(name);
sb.Append(", Age: ");
sb.Append(age);
```

> Prior to .NET 6, `sb.Append($"...")` would first allocate a full interpolated string, then copy it — defeating the purpose.

---

## StringBuilder vs String Concatenation — When to Use Which

| Scenario | Best Approach |
|---|---|
| 2-3 fixed parts | `+` or `$""` interpolation |
| Known at compile time | `$""` with interpolated string handler |
| Loop with many iterations | `StringBuilder` |
| Conditional/branching string assembly | `StringBuilder` |
| Building large documents (HTML, SQL, logs) | `StringBuilder` |
| Single expression, no loop | `string.Concat` / `string.Join` / `$""` |

**Rule of thumb:** if you're concatenating in a loop or conditionally assembling a string with more than ~4 operations, use `StringBuilder`. For everything else, modern C# interpolation is likely good enough or better.

---

## Reusing a StringBuilder

Reuse avoids repeated allocation of the internal buffer:

```csharp
var sb = new StringBuilder(256);

foreach (var item in items)
{
    sb.Clear();                     // resets length, keeps buffer
    sb.Append("Processed: ");
    sb.Append(item.Name);
    logger.Log(sb.ToString());
}
```

`Clear()` sets `Length = 0` but does **not** deallocate. The same buffer is reused across iterations.

For object pooling in high-throughput scenarios:

```csharp
// Using ObjectPool<StringBuilder> from Microsoft.Extensions.ObjectPool
var pool = new DefaultObjectPoolProvider().CreateStringBuilderPool();

var sb = pool.Get();
try
{
    sb.Append("pooled");
    return sb.ToString();
}
finally
{
    pool.Return(sb);  // buffer goes back to the pool
}
```

---

## Thread Safety

StringBuilder is **NOT thread-safe**. Concurrent writes will corrupt the internal state.

Options:
- Use separate StringBuilder instances per thread
- Protect with a `lock` (defeats the purpose in most cases)
- Use `string.Create()` or `IBufferWriter<char>` for lock-free parallel scenarios

---

## Capacity Management

```csharp
var sb = new StringBuilder();
sb.Capacity;    // 16 (default)

sb.Append(new string('x', 100));
sb.Capacity;    // >= 100 (auto-expanded)

// Pre-allocate to avoid resizing
var sb2 = new StringBuilder(10000);

// EnsureCapacity (.NET 6+)
sb.EnsureCapacity(5000);  // guarantees at least 5000 capacity

// MaxCapacity — hard limit, throws if exceeded
var sb3 = new StringBuilder(16, 1000);  // initial 16, max 1000
sb3.Append(new string('x', 1001));      // throws ArgumentOutOfRangeException
```

---

## Common Patterns

### Building CSV

```csharp
var sb = new StringBuilder();
foreach (var row in data)
{
    sb.AppendJoin(',', row.Fields);
    sb.AppendLine();
}
```

### Building SQL (non-parameterized, for logging only)

```csharp
var sb = new StringBuilder("SELECT * FROM Users WHERE 1=1");
if (name != null)
    sb.Append($" AND Name = '{name}'");
if (age.HasValue)
    sb.Append($" AND Age = {age}");
```

### Removing Trailing Separator

```csharp
var sb = new StringBuilder();
foreach (var item in items)
{
    sb.Append(item);
    sb.Append(", ");
}
if (sb.Length >= 2)
    sb.Length -= 2;  // remove trailing ", "

// Or better: use AppendJoin
var sb2 = new StringBuilder();
sb2.AppendJoin(", ", items);
```

---

## Common Pitfalls

| Pitfall | Fix |
|---|---|
| Using StringBuilder for 2-3 simple concatenations | Just use `+` or `$""` — StringBuilder has overhead too |
| Calling `sb.ToString()` inside a loop to check content | Use `sb.Length`, `sb[i]`, or `GetChunks()` instead |
| Not pre-sizing when the final length is roughly known | Pass estimated capacity to constructor |
| Using `sb.Append($"...")` on .NET 5 or earlier | Allocates a temp string — use separate `Append` calls |
| Assuming `sb[i]` is O(1) | It walks the chunk list — fine usually, but avoid tight loops on index |
| Sharing across threads | StringBuilder is not thread-safe — use one per thread |
| Calling `Insert(0, ...)` repeatedly | O(n) per call — build in order or use a `List<string>` and reverse |
