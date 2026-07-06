---
tags:
 - csharp
 - basics
 - types
---

C# 8.0 introduced two struct types — `System.Index` and `System.Range` — that provide first-class support for indexing from the end of a sequence and slicing subsequences.

## System.Index

`Index` represents a position in a sequence. It can count from the start (regular index) or from the end (using the `^` operator).

```csharp
Index first = 0;       // same as new Index(0, fromEnd: false)
Index last  = ^1;      // same as new Index(1, fromEnd: true)
```

The `^` (hat) operator creates an `Index` that counts from the end:
- `^1` — last element
- `^2` — second to last
- `^0` — one past the end (equal to `.Length`, not a valid element)

```csharp
int[] numbers = { 10, 20, 30, 40, 50 };

int a = numbers[^1];   // 50
int b = numbers[^3];   // 30

// Index is a real type — you can store it in a variable
Index i = ^2;
int c = numbers[i];    // 40
```

## System.Range

`Range` represents a start and end `Index` pair, defining a slice of a sequence. Created with the `..` operator.

```csharp
Range r = 1..4;        // same as new Range(new Index(1), new Index(4))
```

The range is **inclusive at the start, exclusive at the end** (like Python slicing):

```csharp
int[] numbers = { 10, 20, 30, 40, 50 };

int[] slice1 = numbers[1..4];    // { 20, 30, 40 }
int[] slice2 = numbers[..3];     // { 10, 20, 30 }       — start defaults to 0
int[] slice3 = numbers[2..];     // { 30, 40, 50 }       — end defaults to ^0
int[] slice4 = numbers[..];      // { 10, 20, 30, 40, 50 } — full copy
int[] slice5 = numbers[^3..^1];  // { 30, 40 }           — mixing ^ with ..

// Range is a real type — you can store it in a variable
Range r = 1..^1;
int[] middle = numbers[r];       // { 20, 30, 40 }
```

## Works With

Both `Index` and `Range` work with any type that has:
- An `int Length` or `int Count` property
- An indexer that takes `int`, `Index`, or `Range`

Built-in support: `arrays`, `string`, `Span<T>`, `ReadOnlySpan<T>`.

```csharp
string text = "Hello, World!";
string word = text[7..12];     // "World"
char last = text[^1];          // '!'
```

## Span Slicing (No Copy)

When used with `Span<T>` or `ReadOnlySpan<T>`, slicing creates a **view** over the original memory — no allocation or copy:

```csharp
Span<int> span = stackalloc int[] { 1, 2, 3, 4, 5 };
Span<int> slice = span[1..4];  // view into the same memory — no copy
```

With arrays, slicing **does** create a new array copy.
