---
tags:
 - csharp
 - basics
 - memory
---

## Overview

The CLR manages two primary regions of memory:

- **Stack** — fast, small, automatic lifetime (scoped to method calls)
- **Heap** — large, flexible, garbage-collected lifetime

Every .NET application uses both. Understanding which data goes where is fundamental to understanding performance, boxing, closures, `ref` returns, `Span<T>`, and `struct` vs `class` decisions.

---

## The Stack

### What It Is

- A **contiguous block of memory** allocated per thread (default ~1 MB on Windows, ~8 MB on Linux)
- Operates as a **LIFO** (last-in, first-out) data structure
- Managed by the CPU — `push` and `pop` are single instructions
- **No garbage collection** — memory is reclaimed instantly when a method returns

### What Lives on the Stack

- **Local variables** of value types (`int`, `double`, `bool`, `struct`, etc.)
- **Method parameters** (value type parameters are copied onto the stack)
- **Return addresses** — where to continue after a method returns
- **References** (pointers) to heap objects — the reference itself is on the stack, the object it points to is on the heap

### How It Works — Stack Frames

Each method call creates a **stack frame**:

```
┌─────────────────────────┐  ← Stack top (grows downward)
│  Method C locals         │  ← Current frame
│  Return address → B      │
├─────────────────────────┤
│  Method B locals         │
│  Return address → A      │
├─────────────────────────┤
│  Method A locals         │
│  Return address → Main   │
├─────────────────────────┤
│  Main() locals           │
└─────────────────────────┘  ← Stack bottom
```

When Method C returns, its entire frame is popped — all its locals are gone instantly. No GC involved.

```csharp
void Calculate()
{
    int x = 10;          // x is on the stack
    double y = 3.14;     // y is on the stack
    Point p;             // struct — all fields on the stack
    p.X = 1;
    p.Y = 2;
}   // x, y, p are all gone — stack frame is popped
```

### Stack Characteristics

| Property | Detail |
|---|---|
| Speed | Extremely fast — pointer increment/decrement |
| Size | Small (~1 MB default per thread) |
| Lifetime | Deterministic — scoped to method execution |
| Thread safety | Each thread has its own stack — inherently isolated |
| Fragmentation | None — always contiguous |
| Overflow | `StackOverflowException` (usually infinite recursion) |

---

## The Heap (Managed Heap)

### What It Is

- A large pool of memory shared across all threads in the process
- Managed by the **Garbage Collector (GC)**
- Objects are allocated at the next available position (bump-pointer allocation for Gen 0)
- Memory is reclaimed non-deterministically when the GC runs

### What Lives on the Heap

- **Reference type instances** — `class`, `string`, `array`, `delegate`, `object`
- **Boxed value types** — when a value type is cast to `object` or an interface
- **Captured variables** in closures (lambdas/anonymous methods that capture locals)
- **Static fields** (stored in a special area associated with the type's metadata)

```csharp
void Example()
{
    int x = 42;                    // stack
    string name = "hello";         // reference on stack, string object on heap
    int[] arr = new int[10];       // reference on stack, array on heap
    var person = new Person();     // reference on stack, Person object on heap
    object boxed = x;             // reference on stack, boxed int on heap
}
```

### Heap Generations

The GC organizes the heap into **generations** based on object age:

```
┌──────────────────────────────────────────────┐
│  Gen 0  │   Gen 1   │       Gen 2            │
│ (short) │  (medium) │       (long)           │
│  ~256KB │   ~2MB    │    grows as needed     │
└──────────────────────────────────────────────┘
         ▲
         └── New objects are allocated here (Gen 0)
```

| Generation | Contains | Collected |
|---|---|---|
| Gen 0 | Newly allocated objects | Most frequently (fast) |
| Gen 1 | Objects that survived one Gen 0 collection | Less frequently |
| Gen 2 | Long-lived objects | Rarely (expensive — full GC) |

**Generational hypothesis:** most objects die young. Gen 0 collections are fast because they only scan a small region.

### Large Object Heap (LOH)

Objects >= 85,000 bytes go directly to the **Large Object Heap**:

- Treated as Gen 2 (collected infrequently)
- **Not compacted by default** (can cause fragmentation)
- `GCSettings.LargeObjectHeapCompactionMode` can enable compaction on next full GC
- Common culprits: large arrays, large strings

```csharp
// This goes on the LOH
var largeArray = new byte[85000];

// This stays on the regular heap (SOH)
var smallArray = new byte[84999];
```

### Pinned Object Heap (POH) — .NET 5+

For objects that must not move in memory (interop, async I/O buffers):

```csharp
// Allocated on POH — never moved by GC, no pinning overhead
byte[] buffer = GC.AllocateArray<byte>(4096, pinned: true);
```

---

## Stack vs Heap — Side by Side

| Aspect | Stack | Heap |
|---|---|---|
| Speed | Very fast (pointer move) | Slower (allocation + GC) |
| Size | Small (~1 MB/thread) | Large (process memory) |
| Lifetime | Deterministic (scope-based) | Non-deterministic (GC) |
| Thread | Private per thread | Shared across threads |
| Fragmentation | None | Possible (especially LOH) |
| Managed by | CPU / CLR automatically | Garbage Collector |
| Contains | Value types, references, frames | Objects, arrays, strings |
| Overflow | `StackOverflowException` | `OutOfMemoryException` |

---

## The Nuances — It's Not "Value Types = Stack, Reference Types = Heap"

This is the most common misconception. The reality is more nuanced:

### Value types that live on the HEAP

**1. Fields of a class:**

```csharp
class Person
{
    public int Age;     // int is a value type, but it lives on the heap
}                       // because Person (a class) is on the heap
// Age is embedded inside the Person object's heap memory
```

**2. Boxed value types:**

```csharp
int x = 42;
object o = x;   // boxing — a new object is allocated on the heap
                 // containing a copy of x's value
```

**3. Captured variables in closures:**

```csharp
void Example()
{
    int counter = 0;   // would normally be on the stack
    Action increment = () => counter++;
    // counter is "captured" — the compiler hoists it into a
    // heap-allocated class (the closure/display class)
}

// Compiler generates something like:
class <>c__DisplayClass
{
    public int counter;   // now on the heap
}
```

**4. Elements of an array (the array itself is on the heap):**

```csharp
int[] numbers = new int[100];
// The array object is on the heap
// Each int element is embedded within the array's heap memory
```

**5. Async method locals:**

```csharp
async Task DoWorkAsync()
{
    int x = 10;        // hoisted to the state machine object on the heap
    await Task.Delay(1000);
    Console.WriteLine(x);  // x must survive across the await
}
// The compiler generates a state machine struct that gets boxed to the heap
```

**6. Iterator locals (yield return):**

```csharp
IEnumerable<int> GetNumbers()
{
    int i = 0;           // hoisted to heap (state machine)
    while (i < 10)
        yield return i++;
}
```

### Reference type data that lives on the STACK

**1. `ref struct` types (Span, ReadOnlySpan):**

```csharp
Span<int> span = stackalloc int[10];
// span is a ref struct — guaranteed to live only on the stack
// cannot be boxed, cannot be a field in a class, cannot be captured
```

**2. `stackalloc` memory:**

```csharp
Span<byte> buffer = stackalloc byte[256];
// 256 bytes allocated directly on the stack — no GC involvement
// Automatically freed when the method returns
```

### The Accurate Rule

> **Where a variable lives depends on its lifetime requirements, not just its type.**
> - Local value types with no captured references → stack
> - Value types that must outlive the current frame → heap (boxing, closures, async, fields of classes)
> - `ref struct` → always stack
> - Everything else with `new` on a reference type → heap

---

## Boxing and Unboxing

The primary performance bridge between stack and heap:

```csharp
int x = 42;           // stack

// Boxing: value type → heap object
object boxed = x;     // allocates on heap, copies value into the box
// Cost: heap allocation + copy

// Unboxing: heap object → value type
int y = (int)boxed;   // copies value back from heap to stack
// Cost: type check + copy

// Hidden boxing — easy to miss:
int n = 10;
Console.WriteLine("Value: " + n);   // n.ToString() avoids boxing here
                                     // but older patterns box implicitly

// Interface dispatch on value types boxes (unless constrained generic)
IComparable c = 42;   // boxed
```

---

## `stackalloc` — Allocating on the Stack Explicitly

```csharp
// Allocate raw memory on the stack (no GC, no heap)
Span<int> numbers = stackalloc int[100];

// Useful for small, short-lived buffers
Span<char> buffer = stackalloc char[256];
input.AsSpan().CopyTo(buffer);

// With initializer (C# 12+)
Span<int> values = stackalloc[] { 1, 2, 3, 4, 5 };

// Guard against stack overflow with a threshold
const int StackThreshold = 256;
Span<byte> buf = length <= StackThreshold
    ? stackalloc byte[length]
    : new byte[length];   // fall back to heap for large sizes
```

**Constraints:**
- Only in `unsafe` context or with `Span<T>` (safe)
- Cannot return the allocated memory from the method
- Risk of `StackOverflowException` if you allocate too much
- Only for unmanaged types or value types without references

---

## Practical Impact on Performance

### Allocation Cost

```
Stack allocation:  ~1 CPU instruction (move stack pointer)
Heap allocation:   ~30-50 instructions (bump pointer + card table + possible GC trigger)
```

### GC Pressure

Every heap allocation is a future GC burden. Reducing allocations:

```csharp
// BAD — allocates a new array every call
int[] GetDefaults() => new int[] { 1, 2, 3 };

// GOOD — single static allocation
private static readonly int[] Defaults = { 1, 2, 3 };
int[] GetDefaults() => Defaults;

// GOOD — stack allocation for temporary work
Span<int> temp = stackalloc int[3] { 1, 2, 3 };
```

### Struct vs Class Decision

```csharp
// Use struct (stack-friendly) when:
// - Small (< ~16 bytes is a common guideline)
// - Short-lived
// - Immutable or logically a value
// - Frequently created in hot paths
struct Point { public float X, Y; }           // 8 bytes, good struct

// Use class (heap) when:
// - Large
// - Long-lived or shared
// - Needs inheritance
// - Mutable with shared identity
class Connection { /* state, handles, etc. */ }
```

---

## Memory Layout Example

```csharp
class Order
{
    public int Id;              // 4 bytes (embedded in heap object)
    public decimal Total;       // 16 bytes (embedded in heap object)
    public string Customer;     // 8 bytes (reference to another heap object)
}

void Process()
{
    int count = 5;              // 4 bytes on stack
    var order = new Order();    // 8 bytes on stack (reference)
                                // + Order object on heap:
                                //   16 bytes header
                                //   + 4 (Id) + 16 (Total) + 8 (Customer ref)
    decimal tax = 0.1m;         // 16 bytes on stack
}
```

```
STACK (Process frame)           HEAP
┌─────────────────────┐        ┌──────────────────────────┐
│ count: 5   (4 bytes)│        │ [Object Header] 16 bytes │
│ order: ────────────────────► │ Id: 0          4 bytes   │
│ tax: 0.1  (16 bytes)│        │ Total: 0.0    16 bytes   │
└─────────────────────┘        │ Customer: ─────────────────► "..."
                               └──────────────────────────┘
```

---

## Diagnosing Stack vs Heap Issues

### Stack Overflow

```csharp
// Infinite recursion — most common cause
void Oops() => Oops();  // StackOverflowException

// Very large stackalloc
Span<byte> huge = stackalloc byte[1_000_000];  // dangerous

// Deep but finite recursion on large structs
void Recursive(BigStruct s, int depth) { ... }
```

### Excessive Heap Allocations (GC Pressure)

Tools to diagnose:
- **dotnet-counters** — watch `gc-heap-size` and `alloc-rate` in real time
- **dotnet-trace** — capture allocation events
- **BenchmarkDotNet** with `[MemoryDiagnoser]` — measures allocations per operation
- **Visual Studio Diagnostic Tools** — allocation profiler

```csharp
// BenchmarkDotNet example
[MemoryDiagnoser]
public class AllocBenchmarks
{
    [Benchmark]
    public string WithConcat()
    {
        string s = "";
        for (int i = 0; i < 100; i++) s += i;
        return s;
    }

    [Benchmark]
    public string WithStringBuilder()
    {
        var sb = new StringBuilder();
        for (int i = 0; i < 100; i++) sb.Append(i);
        return sb.ToString();
    }
}
// Result: WithConcat allocates ~50x more memory
```

---

## Summary of What Goes Where

| Data | Location | Notes |
|---|---|---|
| Local value types | Stack | Unless captured, boxed, or in async/iterator |
| Method parameters (value) | Stack | Copied onto the frame |
| Method parameters (`ref`/`in`/`out`) | Stack | Passed as pointer, no copy |
| Local reference variables | Stack | The reference itself (8 bytes on x64) |
| Reference type instances (`new`) | Heap | Collected by GC |
| String literals | Heap | Interned, never collected |
| Arrays | Heap | Even `int[]` — the array object is a reference type |
| Boxed value types | Heap | `object o = 42;` |
| Static fields | Heap | Special high-frequency area, lives as long as the type |
| Closure-captured locals | Heap | Hoisted into a compiler-generated class |
| Async method state | Heap | State machine is boxed to survive awaits |
| `stackalloc` buffers | Stack | Explicit stack allocation via `Span<T>` |
| `ref struct` (Span, etc.) | Stack | Cannot escape to the heap |
| Large objects (>= 85KB) | LOH | Gen 2, not compacted by default |
