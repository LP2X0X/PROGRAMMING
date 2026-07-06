---
tags:
 - csharp
 - collections
 - generics
---

- The first issue is that using the System.Collections and System.Collections.Specialized classes can result in some poorly performing code, especially when you are manipulating numerical data (e.g., value types). As you’ll see momentarily, the CoreCLR must perform a number of memory transfer operations when you store structures in any nongeneric collection class prototyped to operate on System.Objects, which can hurt runtime execution speed.
- The second issue is that most of the nongeneric collection classes are not type-safe because (again) they were developed to operate on System.Objects, and they could therefore contain anything at all. If a developer needed to create a highly type-safe collection (e.g., a container that can hold objects implementing only a certain interface), the only real choice was to create a new collection class by hand. Doing so was not too labor intensive, but it was a tad on the tedious side.

--- 

Non-generic collections in .NET can lead to performance issues primarily due to type safety and boxing/unboxing overhead. Here’s a deeper look into why non-generic collections have these performance issues:

### 1. **Type Safety**

Non-generic collections store items as `object` types. This lack of type specificity introduces several problems:

- **Runtime Type Checking:**
  - Since non-generic collections store items as `object`, the compiler cannot enforce type safety. This means type mismatches are not caught until runtime, potentially leading to runtime errors.

### 2. **Boxing and Unboxing**

Non-generic collections often cause boxing and unboxing operations, especially when dealing with value types (e.g., `int`, `float`, `structs`):

- **Boxing:**
  - When a value type is added to a non-generic collection, it is "boxed" into an `object`. Boxing is the process of converting a value type to a reference type, which involves creating a new object on the heap and copying the value into it.
  
- **Unboxing:**
  - When retrieving the value, it must be "unboxed" back to its original type. Unboxing is the process of extracting the value type from the object, which involves type checking and copying the value back to a value type variable.
  
- **Performance Impact:**
  - Boxing and unboxing are computationally expensive operations. They increase the workload of the garbage collector since each boxed value type creates a new object on the heap.
  - Frequent boxing and unboxing can significantly degrade performance, especially in performance-critical applications.

### 3. **Type Casting**

Non-generic collections require type casting when retrieving items, which adds overhead and risk:

- **Casting Overhead:**
  - Casting from `object` back to the specific type involves additional processing, including type checking.
  - Incorrect casts can lead to `InvalidCastException`, which must be handled appropriately, adding complexity and potential performance costs due to exception handling.

### Mixing Types — The Silent Danger

Because everything is `object`, the compiler lets you put completely unrelated types in the same collection. The bug only surfaces at runtime:

```csharp
ArrayList list = new ArrayList();
list.Add(1);
list.Add("hello");
list.Add(true);     // no complaint from compiler

// Later when you retrieve...
int num = (int)list[1]; // InvalidCastException at RUNTIME — "hello" is not int
```

With generics, the compiler catches it immediately:

```csharp
List<int> list = new List<int>();
list.Add(1);
list.Add("hello");  // Compile error — caught before you even run the app
```

### Memory Layout — What Boxing Actually Looks Like

```
Without boxing (List<int>):
  Stack: [ 1 | 2 | 3 ]  — contiguous, fast, cache-friendly

With boxing (ArrayList):
  Stack: [ ref | ref | ref ]
           ↓     ↓     ↓
  Heap:  [obj 1] [obj 2] [obj 3]  — 3 extra heap allocations
```

Each boxed value means:
- **Extra heap allocation** per item
- **Extra GC pressure** — more objects for the garbage collector to track and clean up
- **Slower access** — pointer indirection instead of reading the value directly

With thousands of items, the cost adds up significantly.

### Example Comparison

#### Non-generic Collection Example
```csharp
ArrayList nonGenericList = new ArrayList();
nonGenericList.Add(10); // Boxing occurs
int value = (int)nonGenericList[0]; // Unboxing and casting
```

#### Generic Collection Example
```csharp
List<int> genericList = new List<int>();
genericList.Add(10); // No boxing
int value = genericList[0]; // No unboxing or casting
```

### Summary — All Problems at a Glance

| Problem | Non-generic | Generic |
|---|---|---|
| Wrong type in collection | Runtime crash (`InvalidCastException`) | Compile error |
| Value type storage | Boxing (heap alloc per item) | Stored directly, no boxing |
| Casting needed on retrieval | Yes, every time | No |
| GC pressure | Higher (boxed objects) | Lower |
| Type mixing | Allowed silently | Prevented by compiler |

### Why .NET Didn't Use Generics Initially

- **Language and Framework Maturity:**
  - Generics were not part of the initial design of C# and the CLR.
  
- **Backward Compatibility:**
  - Maintaining non-generic collections ensured that existing codebases remained functional without requiring significant changes.

### Conclusion

Non-generic collections can cause performance issues due to boxing/unboxing and the lack of compile-time type safety. The introduction of generics in .NET 2.0 addressed these issues, providing more efficient and type-safe collections that are now the standard recommendation for .NET development.