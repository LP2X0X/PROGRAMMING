---
tags:
 - csharp
 - basics
 - methods
---

## Default Behavior (No Modifier)

The default way a parameter is sent into a function is **by value**. A copy of the data is passed into the function. What gets copied depends on the type:

- **Value type** (int, struct, etc.) — the entire value is copied. Changes inside the method do not affect the caller.
- **Reference type** (class, array, etc.) — the **reference (address) is copied**, not the object. This means:
  - Modifying the object's **properties** inside the method **affects the caller** (both references point to the same object on the heap)
  - **Reassigning** the parameter (`p = new Person()`) does **not** affect the caller (only the local copy of the reference changes)

```csharp
void Modify(Person p)
{
    p.Name = "Changed";   // caller sees this — same object on the heap
    p = new Person();     // caller does NOT see this — only the local copy is reassigned
}
```

## Parameter Modifiers

### 1. `ref` — Pass by Reference

The `ref` keyword passes the variable itself, not a copy. Any changes to the parameter — including reassignment — affect the caller's variable directly.

For reference types, `ref` is essentially a **reference to the reference** (pointer to a pointer). This is only meaningful when you need to **reassign** the caller's variable from inside the method.

**Rules:**
- The variable **must be initialized** before being passed
- Both the caller and the method **must use** the `ref` keyword

```csharp
// Value type with ref
void DoubleIt(ref int x)
{
    x = x * 2;  // caller's variable is modified directly
}

int value = 5;
DoubleIt(ref value);
// value is now 10
```

```csharp
// Reference type with ref — reassignment affects the caller
void Replace(ref Person p)
{
    p.Name = "Bob";       // caller sees this (same as without ref)
    p = new Person();     // caller ALSO sees this — because ref gives access to the reference itself
}

Person someone = new Person { Name = "Alice" };
Replace(ref someone);
// someone now points to the new Person object, not Alice
```

### 2. `out` — Must Assign Before Returning

The `out` keyword is similar to `ref`, but designed for methods that need to **return multiple values**. The key difference is that the method **must assign a value** to the `out` parameter before returning.

**Rules:**
- The variable does **not** need to be initialized before being passed
- The method **must assign** a value before it returns
- Both the caller and the method **must use** the `out` keyword

```csharp
void Divide(int dividend, int divisor, out int quotient, out int remainder)
{
    quotient = dividend / divisor;
    remainder = dividend % divisor;
}

Divide(10, 3, out int q, out int r);  // inline declaration (C# 7+)
// q is 3, r is 1
```

**Common real-world usage** — the TryParse pattern:

```csharp
if (int.TryParse("123", out int result))
{
    Console.WriteLine(result);  // 123
}
```

### 3. `in` — Read-Only Reference

The `in` keyword passes by reference but **prevents modification** inside the method. The compiler enforces that the parameter is read-only.

**Why use it:** Avoids copying large value types (structs) while guaranteeing the method cannot mutate the original.

**Rules:**
- The variable **must be initialized** before being passed
- The method **cannot modify** the parameter (compile-time error if you try)
- The caller can optionally omit the `in` keyword at the call site

```csharp
void PrintLength(in Vector3D v)
{
    // v.X = 10;  // compile error — cannot modify an 'in' parameter
    Console.WriteLine(Math.Sqrt(v.X * v.X + v.Y * v.Y + v.Z * v.Z));
}

Vector3D bigStruct = new Vector3D(1, 2, 3);
PrintLength(in bigStruct);   // no copy made, but method can only read
PrintLength(bigStruct);      // also valid — 'in' is optional at call site
```

### 4. `params` — Variable Number of Arguments

The `params` keyword allows a method to accept a **variable number of identically typed arguments** as an array.

**Rules:**
- Must be the **last parameter** in the method signature
- Only **one** `params` parameter is allowed per method
- The caller can pass a comma-separated list, an array, or nothing at all

```csharp
int Sum(params int[] numbers)
{
    int sum = 0;
    foreach (int num in numbers)
    {
        sum += num;
    }
    return sum;
}

Sum(1, 2, 3, 4, 5);         // 15 — comma-separated
Sum(new int[] { 1, 2, 3 }); // 6  — explicit array
Sum();                       // 0  — no arguments (empty array)
```

## Summary Table

| Modifier | Must Initialize? | Method Must Assign? | Caller Sees Reassignment? | Use Case |
|----------|:-:|:-:|:-:|---|
| *(none)* | Yes | No | No | Default — safe copy |
| `ref` | Yes | No | Yes | Modify caller's variable |
| `out` | No | Yes | Yes | Return multiple values |
| `in` | Yes | No | N/A (read-only) | Avoid copying large structs |
| `params` | N/A | N/A | N/A | Variable argument count |