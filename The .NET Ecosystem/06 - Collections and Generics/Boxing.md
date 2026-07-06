---
tags: 
 - csharp
 - term
 - collections
 - generics
---

## Definition

Boxing is the process of explicitly assigning a **value type** to a `System.Object` variable. The CLR allocates a new object on the heap, copies the value into it, and returns a reference to that heap object.

**Unboxing** is the reverse — extracting the value type back from the heap object, which involves a type check and a copy back to the stack.

> [!important]
> Boxing only applies to **value types** (`int`, `bool`, `double`, `struct`, `enum`, etc.). Reference types are already objects on the heap — assigning them to `object` just copies the reference. No boxing occurs.

---

## How It Works

```csharp
int x = 42;        // lives on the stack
object obj = x;    // boxing happens here
int y = (int)obj;  // unboxing happens here
```

```
BOXING:   value copied from stack → new object on heap → reference returned

Stack:                Heap:
┌──────────┐         ┌─────────────────┐
│ x = 42   │         │ [object header]  │
│ obj = ref┼────────>│   42             │
└──────────┘         └─────────────────┘

UNBOXING: type checked → value copied from heap → back to stack

Stack:                Heap:
┌──────────┐         ┌─────────────────┐
│ x = 42   │         │ [object header]  │
│ obj = ref┼────────>│   42             │
│ y = 42   │         └─────────────────┘
└──────────┘
```

---

## Value Types vs Reference Types — No Boxing for Reference Types

```csharp
// Value types — boxing happens
int x = 42;
object obj = x;         // BOXING: copy 42 to a new heap object

bool b = true;
object obj2 = b;        // BOXING

struct Point { public int X, Y; }
Point p = new Point();
object obj3 = p;        // BOXING

// Reference types — no boxing, ever
string s = "hello";
object obj4 = s;         // NO boxing — just copies the reference

Person p = new Person();
object obj5 = p;          // NO boxing — already on the heap
```

---

## Why Boxing Matters — The Non-Generic Collection Problem

Non-generic collections (`ArrayList`, `Hashtable`, etc.) store everything as `object`. When you put value types in them, every item gets boxed:

```csharp
// ArrayList — internal array is object[]
// Each int gets boxed into a separate heap object
ArrayList list = new ArrayList();
list.Add(42);   // boxing
list.Add(7);    // boxing

Stack:              Heap:
┌────────┐         ┌──────────────────────┐
│ list ref┼───────>│ object[] array        │
└────────┘         │  [0] ref ──> [obj 42] │  ← boxed, separate heap object
                   │  [1] ref ──> [obj 7]  │  ← boxed, separate heap object
                   └──────────────────────┘
```

---

## How Generics Eliminate Boxing

Generic collections like `List<int>` create a **typed array** (`int[]`) internally, so values are stored directly — `object` is never involved:

```csharp
// List<int> — internal array is int[]
// Values stored directly inside the array, no boxing
List<int> list = new List<int>();
list.Add(42);   // no boxing
list.Add(7);    // no boxing

Stack:              Heap:
┌────────┐         ┌──────────────┐
│ list ref┼───────>│ int[] array  │
└────────┘         │  [0] = 42   │  ← value directly in the array
                   │  [1] = 7    │  ← value directly in the array
                   └──────────────┘
```

Both arrays live on the heap (all arrays do), but:
- **`object[]`** — holds references to individually boxed objects, each a separate heap allocation
- **`int[]`** — holds actual values inline, no extra heap objects

---

## Performance Cost of Boxing

Each box/unbox means:
- **Extra heap allocation** per item
- **Extra GC pressure** — more objects for the garbage collector to track and clean up
- **Slower access** — pointer indirection instead of reading the value directly

With thousands of items, the cost adds up significantly.

> [!tip]
> If you're storing value types in a collection, always use a generic collection (`List<int>`, `HashSet<double>`, etc.) to avoid boxing entirely.
