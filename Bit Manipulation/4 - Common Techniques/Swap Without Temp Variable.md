---
tags:
  - bit-manipulation
  - technique
  - xor-swap
---

## Swap Without Temp Variable

The XOR swap algorithm is a classic bit manipulation trick that exchanges two values without using a temporary variable. While historically important, it is largely a curiosity today -- the standard temp-variable swap is both faster and safer on modern hardware.

---

## The XOR Swap Algorithm

```
a ^= b;    // Step 1: a = a XOR b
b ^= a;    // Step 2: b = b XOR (a XOR b) = a
a ^= b;    // Step 3: a = (a XOR b) XOR a = b
```

Three XOR operations, no temporary storage.

---

## Step-by-Step Walkthrough

**Swap a = 12 and b = 7:**

```
Initial state:
  a = 00001100    (12)
  b = 00000111    (7)

Step 1: a ^= b
  a = 00001100
    ^ 00000111
    ----------
  a = 00001011    (11)         a now holds a XOR b
  b = 00000111    (7)          b unchanged

Step 2: b ^= a
  b = 00000111
    ^ 00001011
    ----------
  b = 00001100    (12)         b now holds original a!
  a = 00001011    (11)         a still holds a XOR b

Step 3: a ^= b
  a = 00001011
    ^ 00001100
    ----------
  a = 00000111    (7)          a now holds original b!
  b = 00001100    (12)         b holds original a

Final state:
  a = 00000111    (7)          was 12
  b = 00001100    (12)         was 7
  Swap complete!
```

---

## Why It Works

The proof relies on three properties of the [[XOR Operator]]:

1. **Self-inverse:** `x ^ x = 0` (any value XOR itself is zero)
2. **Identity:** `x ^ 0 = x` (any value XOR zero is itself)
3. **Associativity:** `(x ^ y) ^ z = x ^ (y ^ z)` (grouping does not matter)

Let `A` and `B` be the original values:

```
After Step 1:  a = A ^ B,       b = B
After Step 2:  a = A ^ B,       b = B ^ (A ^ B) = A
After Step 3:  a = (A ^ B) ^ A = B,   b = A
```

**Detailed Step 2 proof:**

```
b = B ^ (A ^ B)
  = B ^ A ^ B          (remove parentheses -- associativity)
  = A ^ B ^ B          (reorder -- commutativity)
  = A ^ 0              (B ^ B = 0 -- self-inverse)
  = A                  (A ^ 0 = A -- identity)
```

**Detailed Step 3 proof:**

```
a = (A ^ B) ^ A
  = A ^ B ^ A          (remove parentheses)
  = A ^ A ^ B          (reorder -- commutativity)
  = 0 ^ B              (A ^ A = 0)
  = B                  (0 ^ B = B)
```

---

## The Self-Swap Trap

```ad-warning
title: Critical Bug -- Same Variable / Same Memory Location
If `a` and `b` refer to the **same variable** or the **same memory address**, the XOR swap destroys the value:

~~~
// a and b point to the same location
a ^= b;    // a = a ^ a = 0      (a is now ZERO)
b ^= a;    // b = 0 ^ 0 = 0      (still zero)
a ^= b;    // a = 0 ^ 0 = 0      (still zero)
// Both are now zero -- value is LOST!
~~~

This commonly happens when:
- Swapping `arr[i]` and `arr[j]` without checking `i != j`
- Swapping through two pointers/references that alias the same object

**Always guard with an equality check:**
~~~
if (&a != &b) { a ^= b; b ^= a; a ^= b; }
~~~
```

---

## Caveats and Why You Should (Almost) Never Use This

```ad-warning
title: Prefer a Temp Variable
The XOR swap is a clever trick, but it is inferior to the standard temp swap in virtually every modern scenario:

1. **Not faster** -- Modern CPUs can execute the temp swap in parallel (the load and store are independent). XOR swap has a strict data dependency chain: each step depends on the previous result, preventing pipelining.

2. **Readability** -- `(a, b) = (b, a)` or `temp = a; a = b; b = a;` is instantly understood. XOR swap requires explanation.

3. **Self-swap bug** -- The temp swap works correctly even when a and b are the same variable.

4. **Compiler optimization** -- Modern compilers recognize the temp-swap pattern and may optimize it to a register swap or even a single XCHG instruction. They cannot optimize the XOR swap as easily because of the dependency chain.
```

| Aspect       | Temp Swap                | XOR Swap                   |
| ------------ | ------------------------ | -------------------------- |
| Readability  | Immediately clear        | Requires explanation       |
| Performance  | Pipelineable, often 1 instruction | 3 dependent operations |
| Self-swap    | Works correctly          | Destroys the value         |
| Extra space  | 1 temp variable (register) | None                     |

---

## Historical Context

The XOR swap was useful in the era when:
- CPU registers were extremely scarce (8-bit microprocessors with few registers)
- Every byte of RAM mattered
- Compilers performed minimal optimization

On modern architectures with dozens of registers and aggressive optimizing compilers, the "saved" temporary variable costs nothing. The XOR swap is now primarily of **educational and interview value**.

```ad-tip
title: Educational Value
Despite its impracticality, XOR swap beautifully demonstrates XOR's algebraic properties (self-inverse, associativity, commutativity) and is a favorite interview question. Understanding *why* it works deepens your grasp of the [[XOR Operator]].
```

---

## Code Examples

### C\#

```csharp
public static class SwapOps
{
    /// <summary>XOR swap -- educational only, prefer tuple swap in practice.</summary>
    public static void XorSwap(ref int a, ref int b)
    {
        if (ReferenceEquals(a, b)) return;  // Not applicable for value types,
                                             // but guard concept shown
        // For value types passed by ref, the addresses could be the same
        // in unsafe code. In safe C#, this is not a concern.
        a ^= b;
        b ^= a;
        a ^= b;
    }

    /// <summary>Standard temp swap -- preferred approach.</summary>
    public static void TempSwap(ref int a, ref int b)
    {
        int temp = a;
        a = b;
        b = temp;
    }

    /// <summary>C# tuple swap -- the most idiomatic approach.</summary>
    public static void TupleSwap(ref int a, ref int b)
    {
        (a, b) = (b, a);
    }
}

// Usage
int x = 42, y = 99;

SwapOps.XorSwap(ref x, ref y);
Console.WriteLine($"x = {x}, y = {y}");  // x = 99, y = 42

SwapOps.TupleSwap(ref x, ref y);
Console.WriteLine($"x = {x}, y = {y}");  // x = 42, y = 99

// Array element swap with guard
int[] arr = { 10, 20, 30 };
int i = 0, j = 2;
if (i != j)
{
    arr[i] ^= arr[j];
    arr[j] ^= arr[i];
    arr[i] ^= arr[j];
}
Console.WriteLine(string.Join(", ", arr));  // 30, 20, 10
```

### C++

```cpp
#include <iostream>
#include <algorithm>  // std::swap

// XOR swap -- educational only
void xorSwap(int& a, int& b) {
    if (&a == &b) return;  // Guard against aliasing
    a ^= b;
    b ^= a;
    a ^= b;
}

// Standard temp swap
void tempSwap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}

int main() {
    int x = 42, y = 99;

    // XOR swap
    xorSwap(x, y);
    std::cout << "After XOR swap: x=" << x << ", y=" << y << "\n";
    // x=99, y=42

    // Standard swap (preferred)
    std::swap(x, y);
    std::cout << "After std::swap: x=" << x << ", y=" << y << "\n";
    // x=42, y=99

    // Demonstrate the self-swap bug
    int z = 50;
    // xorSwap(z, z);  // Would zero out z! Guard prevents it.
    xorSwap(z, z);     // Guard: &a == &b, returns early
    std::cout << "z after self-swap attempt: " << z << "\n";
    // z=50 (preserved by guard)

    return 0;
}
```

### JavaScript

```javascript
// XOR swap -- educational only
function xorSwap(arr, i, j) {
    if (i === j) return;  // Guard against same index
    arr[i] ^= arr[j];
    arr[j] ^= arr[i];
    arr[i] ^= arr[j];
}

// Standard destructuring swap (preferred in JS)
function destructuringSwap(arr, i, j) {
    [arr[i], arr[j]] = [arr[j], arr[i]];
}

// Usage
const arr1 = [10, 20, 30, 40, 50];

// XOR swap elements at index 0 and 4
xorSwap(arr1, 0, 4);
console.log(arr1);  // [50, 20, 30, 40, 10]

// Destructuring swap (preferred)
destructuringSwap(arr1, 1, 3);
console.log(arr1);  // [50, 40, 30, 20, 10]

// For standalone variables, use destructuring:
let a = 42, b = 99;
[a, b] = [b, a];
console.log(`a=${a}, b=${b}`);  // a=99, b=42
```

```ad-note
title: JavaScript XOR Is Integer-Only
JavaScript bitwise operators convert operands to 32-bit signed integers. XOR swap will not work correctly with floats, strings, or values outside the 32-bit signed range. Use destructuring assignment for a universal swap.
```

---

## The Preferred Modern Alternatives

```
C#:         (a, b) = (b, a);               // tuple swap
C++:        std::swap(a, b);                // standard library
JavaScript: [a, b] = [b, a];               // destructuring
```

All three are clearer, faster, and safer than XOR swap.

---

## Related Concepts

- **[[XOR Operator]]** -- the algebraic properties that make XOR swap possible.
- **[[Binary Number System]]** -- understanding bit-level representations.
