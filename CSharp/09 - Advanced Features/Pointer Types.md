---
tags:
 - csharp
 - advanced
 - pointer-types
---

# Pointer Types in C#

## When to Use Pointers

Two common scenarios:
1. **Performance optimization** — directly manipulating memory outside the .NET runtime's management.
2. **Interop with native code** — calling methods in a C-based DLL or COM server that demand pointer parameters. Even here, you can often use `System.IntPtr` and `System.Runtime.InteropServices.Marshal` instead.


---

## The `unsafe` Keyword

Marks a block or member where pointers are allowed, bypassing managed code safety.

**To enable:** add `<AllowUnsafeBlocks>true</AllowUnsafeBlocks>` to your `.csproj` file.

Can be applied to a block, method, struct, or class:

```csharp
unsafe
{
    int number = 10;
    int* ptr = &number;

    Console.WriteLine($"Value: {number}");
    Console.WriteLine($"Address: {(long)ptr:X}");

    *ptr = 20; // modify via pointer
    Console.WriteLine($"New value: {number}"); // 20
}
```

**Use sparingly** — you lose automatic memory management and type safety, and open the door to memory leaks and security issues.


---

## Working with Pointer Types

Use `*` to declare a pointer and `&` to get an address. In C#, `*` binds to the **type**, not each variable name:

```csharp
// C# style — both pi and pj are int pointers
int* pi, pj;
```

### Declaring and Dereferencing

```csharp
unsafe static void PrintValueAndAddress()
{
    int myInt;
    int* ptrToMyInt = &myInt;

    *ptrToMyInt = 123; // assign via pointer indirection

    Console.WriteLine("Value of myInt {0}", myInt);           // 123
    Console.WriteLine("Address of myInt {0:X}", (int)&ptrToMyInt);
}
```

### The `->` Operator (Pointer Member Access)

For struct pointers, use `->` to access members. Alternatively, dereference with `*` and use the dot operator:

```csharp
unsafe static void UsePointerToPoint()
{
    // Using ->
    Point point;
    Point* p = &point;
    p->x = 100;
    p->y = 200;
    Console.WriteLine(p->ToString());

    // Using (*p).member (equivalent)
    Point point2;
    Point* p2 = &point2;
    (*p2).x = 100;
    (*p2).y = 200;
    Console.WriteLine((*p2).ToString());
}
```


---

## The `fixed` Keyword (Pinning)

**Problem:** if you take a pointer to a field on a heap-allocated reference type, the GC could move that object at any time — your pointer becomes invalid.

**Solution:** `fixed` pins the object in memory for the duration of the block.

```csharp
class PointRef
{
    public int x;
    public int y;
    public override string ToString() => $"({x}, {y})";
}

unsafe static void UseAndPinPoint()
{
    PointRef pt = new PointRef { x = 5, y = 6 };

    fixed (int* p = &pt.x) // pt is pinned — GC won't move it
    {
        // safe to use p here
    }
    // pt is unpinned, eligible for GC again

    Console.WriteLine("Point is: {0}", pt);
}
```

The compiler **requires** `fixed` when pointing into a managed object — without it, you get a compile error.


---

## The `stackalloc` Keyword

Allocates a block of memory on the **stack** instead of the heap. Fast to allocate, automatically freed when the method returns, no GC pressure.

```csharp
unsafe void UseStackAlloc()
{
    const int size = 5;
    int* buffer = stackalloc int[size];

    for (int i = 0; i < size; i++)
        buffer[i] = i + 1;

    for (int i = 0; i < size; i++)
        Console.WriteLine($"buffer[{i}] = {buffer[i]}");
}
```

**When to use:**
- Small, short-lived temporary buffers
- Performance-critical code where you want to avoid heap allocation
- Avoiding GC pressure in hot paths

**Watch out for:**
- Stack size is limited (~1 MB default) — don't allocate large buffers
- Requires `unsafe` context (though modern C# allows `Span<T>` with `stackalloc` in safe code)


---

## The `sizeof` Keyword

Returns the size in bytes of an unmanaged type. Works on built-in types anywhere, but requires `unsafe` for custom types:

```csharp
unsafe static void UseSizeOfOperator()
{
    Console.WriteLine("The size of Point is {0}.", sizeof(Point));
}
```
