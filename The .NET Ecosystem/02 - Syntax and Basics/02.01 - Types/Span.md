---
tags:
 - csharp
 - memory
 - performance
 - ref-struct
---

`Span<T>` is a **stack-only view** over a contiguous region of memory. It does not own the memory -- it is a window into memory that already exists somewhere else (an array, a string, a `stackalloc` buffer, or even unmanaged memory). Because it is a [[struct|ref struct]], it can never escape the stack, which makes it safe for the runtime to optimize aggressively.

The key insight: `Span<T>` lets you **read and write slices of memory without allocating anything**. No copies, no new arrays, no GC pressure.

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };
Span<int> span = numbers.AsSpan();   // zero-cost view over the array
span[0] = 99;                         // modifies the original array
Console.WriteLine(numbers[0]);        // 99
```

````ad-summary
**`Span<T>` = a pointer that can reach anywhere, but the pointer itself must stay on the stack.**

```
Stack                          Heap                    Unmanaged
┌──────────────┐
│ Span<byte>   │──────────→  byte[] array
│  ref + len   │
│              │──────────→                           native buffer
│              │──────────→  stackalloc byte[]
└──────────────┘
   must live here              can point to any of these
```

That's what makes it powerful — one unified type for slicing into any kind of memory, with zero allocation. And that's what `ref struct` enforces — the pointer never escapes to the heap where it could outlive what it points to.
````

---

### Span\<T\> vs ReadOnlySpan\<T\>

| | `Span<T>` | `ReadOnlySpan<T>` |
|---|---|---|
| Read elements | Yes | Yes |
| Write elements | Yes | No -- compile error |
| Created from | arrays, `stackalloc`, pointers | arrays, `stackalloc`, pointers, **strings** |
| Use case | Mutating buffers | Reading without modifying (e.g. parsing) |

`ReadOnlySpan<T>` is the read-only counterpart. A `Span<T>` implicitly converts to `ReadOnlySpan<T>`, but not the other way around.

```csharp
Span<int> writable = new int[] { 1, 2, 3 };
ReadOnlySpan<int> readable = writable;  // implicit conversion -- OK
// Span<int> back = readable;           // compile error -- cannot go back

// Strings only work with ReadOnlySpan<char>
ReadOnlySpan<char> chars = "hello world".AsSpan();
// Span<char> mutable = "hello".AsSpan();  // compile error -- strings are immutable
```

> [!ad-note] Why strings only allow ReadOnlySpan
> [[string|Strings]] are immutable in .NET. Allowing a `Span<char>` over a string would let you mutate its contents, breaking interning, thread safety, and every assumption the runtime makes about strings. So the compiler only permits `ReadOnlySpan<char>`.

---

### Creating Spans

#### From an Array

```csharp
int[] array = { 10, 20, 30, 40, 50 };

Span<int> full = array.AsSpan();           // entire array
Span<int> slice = array.AsSpan(1, 3);      // elements at index 1, 2, 3 → {20, 30, 40}

// Implicit conversion also works
Span<int> implicit = array;
```

#### From a String

```csharp
string text = "2026-06-30";
ReadOnlySpan<char> year = text.AsSpan(0, 4);   // "2026" -- no allocation
ReadOnlySpan<char> month = text.AsSpan(5, 2);  // "06"   -- no allocation
```

#### From stackalloc

`stackalloc` allocates memory on the stack. Combined with `Span<T>`, it gives you a temporary buffer with **zero heap allocation**:

```csharp
Span<byte> buffer = stackalloc byte[256];
buffer[0] = 0xFF;
buffer.Fill(0);     // zero the buffer
```

See: [[Stack and Heap]] -- stackalloc memory lives on the stack frame and is freed automatically when the method returns.

#### From Unmanaged Memory

```csharp
IntPtr ptr = Marshal.AllocHGlobal(100);
Span<byte> span = new Span<byte>((void*)ptr, 100);  // unsafe context required
span[0] = 42;
Marshal.FreeHGlobal(ptr);
```

> [!summary] Section Summary
> - `.AsSpan()` is the standard way to create a span from an array or string
> - `stackalloc` + `Span<T>` gives zero-alloc temporary buffers
> - Unmanaged pointers can be wrapped in a span for bounds-checked access

---

### Slicing -- Zero-Copy Subviews

Slicing a span creates a **new span that points to the same underlying memory**. No data is copied.

```csharp
int[] data = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
Span<int> span = data;

Span<int> middle = span.Slice(3, 4);   // {3, 4, 5, 6}
Span<int> tail   = span.Slice(7);      // {7, 8, 9}
Span<int> head   = span[..3];          // {0, 1, 2} -- range syntax
```

#### Range Syntax with Spans

[[Indices and Ranges]] work natively with spans:

```csharp
Span<int> span = new int[] { 0, 1, 2, 3, 4, 5 };

Span<int> a = span[1..4];    // {1, 2, 3}
Span<int> b = span[..3];     // {0, 1, 2}
Span<int> c = span[3..];     // {3, 4, 5}
Span<int> d = span[^2..];    // {4, 5}
```

> [!warning] Slicing does not copy
> Because slicing is zero-copy, modifying a slice modifies the original data:
> ```csharp
> int[] array = { 1, 2, 3, 4, 5 };
> Span<int> slice = array.AsSpan(0, 3);
> slice[0] = 99;
> Console.WriteLine(array[0]); // 99 -- original array was modified
> ```
> This is the point of spans -- but it can surprise you if you forget.

> [!summary] Section Summary
> - `.Slice()` and range syntax (`[1..4]`) create subviews over the same memory
> - Zero-copy means no allocation, but also means mutations propagate to the source
> - Range syntax from [[Indices and Ranges]] is the idiomatic way to slice spans

---

### Common Use Cases

#### 1. Parsing Strings Without Allocations

The classic motivation for `Span<T>`. Compare `Substring` (allocates) vs span slicing (free):

```csharp
// BAD -- every Substring allocates a new string on the heap
string ParseDate_Old(string input)
{
    string year  = input.Substring(0, 4);    // allocation
    string month = input.Substring(5, 2);    // allocation
    string day   = input.Substring(8, 2);    // allocation
    return $"{year}-{month}-{day}";
}

// GOOD -- zero allocations until you actually need a string
void ParseDate_Span(ReadOnlySpan<char> input)
{
    ReadOnlySpan<char> year  = input[..4];     // no allocation
    ReadOnlySpan<char> month = input[5..7];    // no allocation
    ReadOnlySpan<char> day   = input[8..10];   // no allocation

    // int.Parse accepts ReadOnlySpan<char> directly
    int y = int.Parse(year);
    int m = int.Parse(month);
    int d = int.Parse(day);
}
```

#### 2. Processing Array Segments

Pass a "view" of an array to a method without copying:

```csharp
void ProcessChunk(Span<byte> chunk)
{
    for (int i = 0; i < chunk.Length; i++)
        chunk[i] ^= 0xFF;  // in-place transform
}

byte[] data = new byte[1024];
ProcessChunk(data.AsSpan(0, 256));    // no copy
ProcessChunk(data.AsSpan(256, 256));  // no copy
```

#### 3. High-Performance APIs

Many .NET APIs now accept `Span<T>` or `ReadOnlySpan<T>`:

```csharp
Span<char> buffer = stackalloc char[64];
42.TryFormat(buffer, out int charsWritten);  // format into span -- no string alloc

Encoding.UTF8.GetBytes("hello", stackalloc byte[128]);  // encode into span

using var stream = File.OpenRead("data.bin");
int bytesRead = stream.Read(stackalloc byte[512]);       // read into span
```

> [!summary] Section Summary
> - Replace `Substring` with span slicing for zero-allocation string parsing
> - Pass array segments as spans instead of copying into new arrays
> - Modern .NET APIs (Parse, TryFormat, Stream, Encoding) work directly with spans

---

### Ref Struct Restrictions

`Span<T>` is a `ref struct`, which means the runtime guarantees it **never escapes the stack**. This enables performance optimizations but imposes hard restrictions. See [[struct|ref struct]] for the full picture.

#### What You Cannot Do with Span\<T\>

All of these are **compile errors**:

```csharp
class MyClass { Span<int> data; }                       // field in a class
object o = stackalloc int[10];                           // boxing
async Task F() { Span<byte> buf = stackalloc byte[64]; } // async method
Action a = () => span.Length;                             // lambda capture
```

- Cannot be a field in a class or regular struct
- Cannot be boxed to `object` or interface
- Cannot be used in async methods (span could outlive the await)
- Cannot be captured by lambdas (closure is a heap-allocated class)
- Cannot implement interfaces (relaxed in C# 13 for limited cases)

> [!danger] The "why" behind every restriction
> Every restriction exists for the same reason: `Span<T>` points to stack memory. If it escaped to the heap (class field, boxed object, async state machine, lambda closure), it could outlive the stack frame it points to -- resulting in a dangling pointer and memory corruption. The compiler enforces these rules to prevent that.

> [!summary] Section Summary
> - `Span<T>` is a `ref struct` -- it can only live on the stack
> - Cannot be stored in classes, boxed, used in async, or captured by lambdas
> - All restrictions prevent the span from outliving the memory it points to

---

### Memory\<T\> and ReadOnlyMemory\<T\>

When you need a "span-like" view but must store it on the heap or use it in async code, use `Memory<T>` instead. It is the **heap-safe counterpart** to `Span<T>`.

| | `Span<T>` | `Memory<T>` |
|---|---|---|
| Type | `ref struct` (stack-only) | Regular `struct` (can go on heap) |
| Class field / async / boxed | No | Yes |
| Performance | Fastest -- direct pointer | Slight overhead -- call `.Span` |
| Mutability pair | `Span<T>` / `ReadOnlySpan<T>` | `Memory<T>` / `ReadOnlyMemory<T>` |

```csharp
// Memory<T> can live in a class field -- Span<T> cannot
class Buffer
{
    private Memory<byte> _data;
    public Buffer(byte[] source) => _data = source.AsMemory();

    public void Process()
    {
        Span<byte> span = _data.Span;  // convert to Span for hot-path work
        span[0] = 0xFF;
    }
}

// Memory<T> works in async methods -- Span<T> cannot
async Task ProcessAsync(Memory<byte> data)
{
    await Task.Delay(100);
    Span<byte> span = data.Span;  // convert to Span in synchronous sections
    span.Fill(0);
}
```

> [!ad-note] Rule of thumb
> Use `Span<T>` in synchronous, hot-path code where you control the lifetime. Use `Memory<T>` when you need to store the reference, pass it to async methods, or cross API boundaries that don't accept spans.

> [!summary] Section Summary
> - `Memory<T>` is a heap-safe wrapper -- use it when `Span<T>` restrictions are too limiting
> - Call `.Span` on a `Memory<T>` to get a `Span<T>` for high-performance work
> - Choose `Memory<T>` for async code, class fields, and long-lived references

---

### Span vs Array vs ArraySegment -- When to Use Which

**Decision guide:**

1. **Need to own data?** Use `T[]`
2. **Hot synchronous path, temporary view?** Use `Span<T>`
3. **Need to store a view or use in async?** Use `Memory<T>`
4. **Legacy code with ArraySegment?** Migrate to `Memory<T>` when possible

> [!warning] ArraySegment is largely obsolete
> `ArraySegment<T>` predates spans (.NET 2.0). It can only wrap arrays -- not stackalloc or unmanaged memory. New code should use `Span<T>` or `Memory<T>`.

> [!summary] Section Summary
> - `Span<T>` for synchronous hot paths, `Memory<T>` for async/storage
> - `ArraySegment<T>` is legacy -- prefer `Memory<T>` for the same role
> - Raw arrays only when you need to own the data

---

### Relationship to stackalloc

`stackalloc` allocates a block of memory on the stack. Before `Span<T>` existed (pre-C# 7.2), `stackalloc` required `unsafe` code. Now, assigning to a `Span<T>` makes it safe:

```csharp
// Pre-C# 7.2 -- required unsafe
unsafe
{
    int* buffer = stackalloc int[100];
    buffer[0] = 42;
}

// C# 7.2+ -- safe with Span<T>
Span<int> buffer = stackalloc int[100];
buffer[0] = 42;  // bounds-checked, no unsafe keyword needed
```

> [!danger] Stack overflow risk
> The stack is small (typically 1 MB per thread). Allocating large buffers with `stackalloc` can cause a **StackOverflowException** with no recovery. Keep stackalloc buffers small -- a common pattern is to fall back to array allocation for large sizes:
> ```csharp
> const int StackAllocThreshold = 256;
>
> int length = GetRequiredLength();
> Span<byte> buffer = length <= StackAllocThreshold
>     ? stackalloc byte[length]
>     : new byte[length];   // falls back to heap for large buffers
> ```

See: [[Stack and Heap]] for details on stack size limits and allocation.

> [!summary] Section Summary
> - `stackalloc` + `Span<T>` gives safe, zero-alloc temporary buffers without `unsafe`
> - Keep stackalloc small (< 256-512 bytes) to avoid stack overflow
> - Use a threshold pattern to fall back to array allocation for large sizes

---

### Real-World Examples

#### Example 1: Splitting a CSV Line Without Allocations

`string.Split()` allocates an array of new strings. With spans, parse in-place:

```csharp
void ParseCsvLine(ReadOnlySpan<char> line)
{
    while (line.Length > 0)
    {
        int comma = line.IndexOf(',');
        ReadOnlySpan<char> field = comma == -1 ? line : line[..comma];

        ProcessField(field);  // use the field -- no allocation

        line = comma == -1 ? default : line[(comma + 1)..];
    }
}
```

#### Example 2: Trimming and Parsing a Number

```csharp
// Traditional -- allocates an intermediate string
int Parse_Old(string input)
{
    string trimmed = input.Trim();       // allocation
    return int.Parse(trimmed);
}

// Span-based -- zero allocations
int Parse_Span(ReadOnlySpan<char> input)
{
    ReadOnlySpan<char> trimmed = input.Trim();  // no allocation -- returns a sub-span
    return int.Parse(trimmed);                   // int.Parse accepts ReadOnlySpan<char>
}
```

> [!summary] Section Summary
> - Span-based parsing eliminates the string allocations that `Split` and `Substring` produce
> - Many .NET methods (int.Parse, Trim, IndexOf) have span overloads
> - stackalloc + span gives fully stack-allocated processing pipelines

---

### Key Methods on Span\<T\>

| Method | Description |
|---|---|
| `span.Slice(start, length)` | Zero-copy subview |
| `span[range]` | Range-based slicing (see [[Indices and Ranges]]) |
| `span.Fill(value)` | Set all elements to a value |
| `span.Clear()` | Zero all elements |
| `span.CopyTo(destination)` | Copy contents to another span |
| `span.TryCopyTo(destination)` | Copy if destination is large enough; returns bool |
| `span.IndexOf(value)` | Find first occurrence |
| `span.Contains(value)` | Check if value exists |
| `span.Trim()` | Trim whitespace (for `ReadOnlySpan<char>`) |
| `span.StartsWith(other)` | Check prefix |
| `span.SequenceEqual(other)` | Element-by-element equality |
| `span.ToArray()` | Copy to a new array (allocates) |
| `span.ToString()` | For `ReadOnlySpan<char>`, creates a new string (allocates) |

---

### Comprehensive Summary

> [!summary] Complete Summary
> `Span<T>` is a `ref struct` providing a **zero-allocation, bounds-checked view** over contiguous memory. `ReadOnlySpan<T>` is its immutable counterpart for working with [[string|strings]] without copying. Spans eliminate allocations in hot paths -- slicing produces a new span over the same memory with zero copies and zero GC pressure.
>
> The tradeoff: as a [[struct|ref struct]], `Span<T>` can only live on the stack (no class fields, async, lambdas, or boxing). Use `Memory<T>` when you hit these restrictions.
>
> **When to choose:** `Span<T>` for synchronous hot paths; `Memory<T>` for async/storage; `T[]` to own data; `ArraySegment<T>` is legacy. Modern .NET APIs (parsing, formatting, I/O) increasingly accept spans as the foundation of high-performance code.

---

### Related Topics

- [[struct]] -- ref struct is a special kind of struct
- [[string]] -- ReadOnlySpan\<char\> for zero-alloc string processing
- [[StringBuilder]] -- alternative for string building (heap-based)
- [[Indices and Ranges]] -- range syntax works with spans
- [[Stack and Heap]] -- span lives on the stack, stackalloc context
- [[Memory and ReadOnlyMemory]] -- heap-safe counterpart (potential future note)
- [[ArrayPool]] -- reusable array buffers to reduce GC pressure (potential future note)
- [[Unsafe Code]] -- pre-span pointer arithmetic (potential future note)
