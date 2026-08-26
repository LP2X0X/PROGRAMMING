---
tags:
  - bit-manipulation
  - technique
  - popcount
---

## Count Set Bits

Counting the number of `1` bits in an integer is known as **population count** (popcount) or **Hamming weight**. This is one of the most frequently needed bit manipulation operations, appearing in everything from error detection to combinatorial algorithms.

---

## Terminology

| Term             | Meaning                                      |
| ---------------- | -------------------------------------------- |
| Population count | Number of `1` bits in a binary representation |
| Popcount         | Abbreviation of population count              |
| Hamming weight   | Same as popcount -- the number of non-zero positions |

---

## Naive Approach: Loop and Check Each Bit

The simplest method examines every bit one at a time using the [[AND Operator]] and [[Right Shift]]:

```
Algorithm:
  1. Initialize count = 0
  2. While n != 0:
     a. If (n & 1) == 1, increment count   (check the lowest bit)
     b. Right-shift n by 1                  (move to next bit)
  3. Return count
```

```
Walkthrough: n = 0b11010110 (214), expecting 5 set bits

  Step  |   n (binary)   | n & 1 | count
  ------|----------------|-------|------
    1   |  1 1 0 1 0 1 1 0 |   0   |   0
    2   |  0 1 1 0 1 0 1 1 |   1   |   1
    3   |  0 0 1 1 0 1 0 1 |   1   |   2
    4   |  0 0 0 1 1 0 1 0 |   0   |   2
    5   |  0 0 0 0 1 1 0 1 |   1   |   3
    6   |  0 0 0 0 0 1 1 0 |   0   |   3
    7   |  0 0 0 0 0 0 1 1 |   1   |   4
    8   |  0 0 0 0 0 0 0 1 |   1   |   5
  done  |  0 0 0 0 0 0 0 0 |       |   5
```

**Time complexity:** O(W) where W is the bit width (e.g., 32 for a 32-bit int). Every bit is checked regardless of how many are set.

```ad-note
title: Unsigned Right Shift in JavaScript
In JavaScript, use `>>>` (unsigned right shift) instead of `>>` to avoid infinite loops with negative numbers. The `>>` operator preserves the sign bit, so a negative number never becomes zero. See [[Unsigned Right Shift]].
```

---

## Brian Kernighan's Algorithm

A clever optimization that runs in O(k) time, where k is the **number of set bits** (not the total bit width).

**Key insight:** The expression `n & (n - 1)` clears the [[Isolate and Clear Lowest Set Bit|lowest set bit]] of `n`. By repeatedly clearing the lowest set bit and counting iterations, we count exactly the number of `1` bits.

```
Algorithm:
  1. Initialize count = 0
  2. While n != 0:
     a. n = n & (n - 1)    (clear the lowest set bit)
     b. Increment count
  3. Return count
```

### ASCII Art Walkthrough

**n = 0b11010110 (214), expecting 5 set bits:**

```
Iteration 1:
  n       = 11010110
  n - 1   = 11010101
  n&(n-1) = 11010100      lowest set bit (bit 1) cleared
  count = 1                              ^

Iteration 2:
  n       = 11010100
  n - 1   = 11010011
  n&(n-1) = 11010000      lowest set bit (bit 2) cleared
  count = 2                        ^

Iteration 3:
  n       = 11010000
  n - 1   = 11001111
  n&(n-1) = 11000000      lowest set bit (bit 4) cleared
  count = 3                  ^

Iteration 4:
  n       = 11000000
  n - 1   = 10111111
  n&(n-1) = 10000000      lowest set bit (bit 6) cleared
  count = 4            ^

Iteration 5:
  n       = 10000000
  n - 1   = 01111111
  n&(n-1) = 00000000      lowest set bit (bit 7) cleared
  count = 5          ^

  n == 0, stop. Result: 5 set bits.
```

**Why it's better:** For a 32-bit integer with only 3 bits set, the naive approach does 32 iterations while Kernighan's does only 3.

```ad-tip
title: Recognizing n & (n - 1)
The expression `n & (n - 1)` appears everywhere in bit manipulation:
- Here: counting set bits by iteratively clearing them
- In [[Check Power of Two]]: if `n & (n - 1) == 0`, only one bit was set
- In [[Isolate and Clear Lowest Set Bit]]: it is the "clear lowest set bit" operation itself
```

---

## Lookup Table Approach

For maximum speed without hardware support, precompute the popcount for every possible byte value (256 entries) and sum four lookups for a 32-bit integer.

```
table[0x00] = 0
table[0x01] = 1
table[0x02] = 1
table[0x03] = 2
...
table[0xFF] = 8

popcount(n) = table[n & 0xFF]
            + table[(n >> 8) & 0xFF]
            + table[(n >> 16) & 0xFF]
            + table[(n >> 24) & 0xFF]
```

This is O(1) with a small constant (4 table lookups for 32-bit), but the table consumes 256 bytes of memory and may cause cache misses.

---

## Built-in Functions

Modern languages and CPUs provide dedicated popcount instructions:

| Language | Function                          | Header / Namespace         | Notes                          |
| -------- | --------------------------------- | -------------------------- | ------------------------------ |
| C#       | `BitOperations.PopCount(uint)`    | `System.Numerics`          | .NET Core 3.0+ / .NET 5+      |
| C++      | `__builtin_popcount(unsigned)`    | (GCC/Clang built-in)      | Non-standard but ubiquitous    |
| C++      | `std::popcount(unsigned)`         | `<bit>` (C++20)           | Standard, unsigned types only  |
| JavaScript | (none)                          |                            | Use manual implementation      |

```ad-tip
title: Hardware Acceleration
On x86 CPUs with the POPCNT instruction (SSE4.2+), `BitOperations.PopCount` and `__builtin_popcount` compile to a single CPU instruction -- far faster than any software approach. Always prefer built-ins when available.
```

---

## Code Examples

### C\#

```csharp
using System;
using System.Numerics;

public static class PopCount
{
    /// <summary>Naive approach: check each bit.</summary>
    public static int CountBitsNaive(int n)
    {
        // Work with uint to handle negative numbers correctly
        uint value = (uint)n;
        int count = 0;
        while (value != 0)
        {
            count += (int)(value & 1);
            value >>= 1;
        }
        return count;
    }

    /// <summary>Brian Kernighan's algorithm.</summary>
    public static int CountBitsKernighan(int n)
    {
        uint value = (uint)n;
        int count = 0;
        while (value != 0)
        {
            value &= (value - 1);  // Clear lowest set bit
            count++;
        }
        return count;
    }

    /// <summary>Built-in popcount (.NET Core 3.0+).</summary>
    public static int CountBitsBuiltin(int n)
    {
        return BitOperations.PopCount((uint)n);
    }
}

// Usage
int[] values = { 0, 1, 7, 214, 255, -1 };
foreach (int v in values)
{
    int naive = PopCount.CountBitsNaive(v);
    int kern  = PopCount.CountBitsKernighan(v);
    int built = PopCount.CountBitsBuiltin(v);

    Console.WriteLine($"{v,5} -> naive={naive}, kernighan={kern}, builtin={built}");
}
// Output:
//     0 -> naive=0, kernighan=0, builtin=0
//     1 -> naive=1, kernighan=1, builtin=1
//     7 -> naive=3, kernighan=3, builtin=3
//   214 -> naive=5, kernighan=5, builtin=5
//   255 -> naive=8, kernighan=8, builtin=8
//    -1 -> naive=32, kernighan=32, builtin=32
```

### C++

```cpp
#include <iostream>
#include <bit>        // C++20 for std::popcount
#include <cstdint>

// Naive approach
int countBitsNaive(unsigned int n) {
    int count = 0;
    while (n != 0) {
        count += (n & 1);
        n >>= 1;
    }
    return count;
}

// Brian Kernighan's algorithm
int countBitsKernighan(unsigned int n) {
    int count = 0;
    while (n != 0) {
        n &= (n - 1);  // Clear lowest set bit
        count++;
    }
    return count;
}

// Built-in: GCC/Clang
int countBitsBuiltinGcc(unsigned int n) {
    return __builtin_popcount(n);
}

// Built-in: C++20 standard
int countBitsStd(unsigned int n) {
    return std::popcount(n);
}

int main() {
    unsigned int values[] = {0, 1, 7, 214, 255, 0xFFFFFFFF};

    for (auto v : values) {
        std::cout << v << " -> "
                  << "naive=" << countBitsNaive(v) << ", "
                  << "kernighan=" << countBitsKernighan(v) << ", "
                  << "builtin=" << countBitsBuiltinGcc(v) << "\n";
    }

    return 0;
}
```

### JavaScript

```javascript
// Naive approach (use >>> for unsigned right shift)
function countBitsNaive(n) {
    let count = 0;
    // Convert to unsigned 32-bit integer
    n = n >>> 0;
    while (n !== 0) {
        count += (n & 1);
        n >>>= 1;  // unsigned right shift to avoid sign extension
    }
    return count;
}

// Brian Kernighan's algorithm
function countBitsKernighan(n) {
    // Convert to unsigned 32-bit integer
    n = n >>> 0;
    let count = 0;
    while (n !== 0) {
        n &= (n - 1);  // Clear lowest set bit
        count++;
    }
    return count;
}

// Bit-parallel approach (no built-in popcount in JS)
function countBitsFast(n) {
    n = n >>> 0;
    n = n - ((n >> 1) & 0x55555555);
    n = (n & 0x33333333) + ((n >> 2) & 0x33333333);
    n = (n + (n >> 4)) & 0x0F0F0F0F;
    return (n * 0x01010101) >> 24;
}

// Usage
const values = [0, 1, 7, 214, 255, -1];
values.forEach(v => {
    console.log(`${v}: naive=${countBitsNaive(v)}, ` +
                `kernighan=${countBitsKernighan(v)}, ` +
                `fast=${countBitsFast(v)}`);
});
// Output:
// 0: naive=0, kernighan=0, fast=0
// 1: naive=1, kernighan=1, fast=1
// 7: naive=3, kernighan=3, fast=3
// 214: naive=5, kernighan=5, fast=5
// 255: naive=8, kernighan=8, fast=8
// -1: naive=32, kernighan=32, fast=32
```

```ad-warning
title: Signed vs Unsigned in JavaScript
JavaScript has no unsigned integer type. Use `n >>> 0` to reinterpret a signed 32-bit number as unsigned before bit counting. Without this, `countBitsNaive(-1)` would loop forever because `>> 1` preserves the sign bit. See [[Unsigned Right Shift]].
```

---

## Applications

### Hamming Distance

The **Hamming distance** between two integers is the number of bit positions where they differ. It is computed as the popcount of their [[XOR Operator|XOR]]:

```
hammingDistance(a, b) = popcount(a ^ b)

Example:
  a = 10110100
  b = 01101001
  ---------
  a ^ b = 11011101    (5 bits differ)

  Hamming distance = 5
```

### Counting Active Flags

When using [[Flag Enums and Bit Flags]], popcount tells you how many flags are active:

```csharp
[Flags]
enum Permissions { Read = 1, Write = 2, Execute = 4, Admin = 8 }

var perms = Permissions.Read | Permissions.Write | Permissions.Execute;
int activeCount = BitOperations.PopCount((uint)perms);  // 3
```

---

## Algorithm Comparison

| Algorithm          | Time         | Space  | Best When                        |
| ------------------ | ------------ | ------ | -------------------------------- |
| Naive (loop)       | O(W)         | O(1)   | Understanding the concept        |
| Kernighan's        | O(k)         | O(1)   | Few bits set, no hardware popcount |
| Lookup table       | O(1)*        | O(256) | High throughput, no HW support   |
| Built-in / HW      | O(1)         | O(1)   | Always, when available           |

W = bit width, k = number of set bits. *Lookup table is O(W/8) lookups, treated as O(1) for fixed W.

---

## Related Concepts

- **[[Isolate and Clear Lowest Set Bit]]** -- `n & (n - 1)` is the core of Kernighan's algorithm.
- **[[AND Operator]]** -- used in every approach to examine individual bits.
- **[[Right Shift]]** -- used in the naive approach to shift bits into position.
- **[[XOR Operator]]** -- popcount of XOR gives Hamming distance.
- **[[Bit Manipulation in CSharp]]** -- `BitOperations.PopCount` details.
- **[[Bit Manipulation in CPP]]** -- `__builtin_popcount` and `std::popcount` details.
