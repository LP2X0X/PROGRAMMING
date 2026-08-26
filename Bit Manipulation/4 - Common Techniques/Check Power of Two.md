---
tags:
  - bit-manipulation
  - technique
  - power-of-two
---

## Check Power of Two

Determining whether an integer is a power of two is one of the most elegant applications of bit manipulation. A power of two in binary has **exactly one bit set**, which leads to a beautifully simple O(1) check.

---

## The Trick

**Formula:** `n > 0 && (n & (n - 1)) == 0`

This single expression checks:
1. `n > 0` -- zero is not a power of two (and avoids a false positive)
2. `(n & (n - 1)) == 0` -- after clearing the [[Isolate and Clear Lowest Set Bit|lowest set bit]], nothing remains

If a number has exactly one bit set, clearing that bit produces zero. If it has more than one bit set, other bits survive.

---

## Why It Works

A power of 2 in the [[Binary Number System]] has exactly one `1` bit:

```
  1  = 00000001    (2^0)
  2  = 00000010    (2^1)
  4  = 00000100    (2^2)
  8  = 00001000    (2^3)
 16  = 00010000    (2^4)
 32  = 00100000    (2^5)
 64  = 01000000    (2^6)
128  = 10000000    (2^7)
```

When you subtract 1 from a power of 2, the single `1` bit turns to `0`, and all lower bits become `1`:

```
n     = 00001000    (8)
n - 1 = 00000111    (7)
```

ANDing these two values gives zero because they share no `1` bits:

```
    00001000    (n = 8)
  & 00000111    (n - 1 = 7)
  ----------
    00000000    (result = 0  -->  IS a power of 2)
```

---

## ASCII Art Walkthrough

### Case 1: n = 8 (IS a power of 2)

```
n = 8:
  Binary:    0  0  0  0  1  0  0  0
  Bit pos:   7  6  5  4  3  2  1  0
                         ^
                    only one bit set

n - 1 = 7:
  Binary:    0  0  0  0  0  1  1  1
  Bit pos:   7  6  5  4  3  2  1  0
                         ^  ^--^--^
              this bit    all these bits
              cleared     turned ON

n & (n-1):
    00001000
  & 00000111
  ----------
    00000000   == 0   --> Power of 2!
```

### Case 2: n = 6 (NOT a power of 2)

```
n = 6:
  Binary:    0  0  0  0  0  1  1  0
  Bit pos:   7  6  5  4  3  2  1  0
                            ^  ^
                      two bits set (not a power of 2)

n - 1 = 5:
  Binary:    0  0  0  0  0  1  0  1
  Bit pos:   7  6  5  4  3  2  1  0
                            ^     ^
            bit 2 survives   bit 1 cleared, bit 0 set

n & (n-1):
    00000110
  & 00000101
  ----------
    00000100   == 4   != 0   --> NOT a power of 2
                ^
           bit 2 survived the AND -- proves multiple bits were set
```

### Case 3: n = 1 (IS a power of 2: 2^0 = 1)

```
n = 1:
  Binary:    0  0  0  0  0  0  0  1
                                  ^
                           one bit set

n - 1 = 0:
  Binary:    0  0  0  0  0  0  0  0

n & (n-1):
    00000001
  & 00000000
  ----------
    00000000   == 0   --> Power of 2!
```

---

## Edge Cases

| Value          | Result           | Reason                                              |
| -------------- | ---------------- | --------------------------------------------------- |
| `0`            | **Not** power of 2 | Zero has no bits set; `n > 0` guard catches this   |
| `1`            | **Is** power of 2  | `2^0 = 1`; exactly one bit set                    |
| `-1`           | **Not** power of 2 | Negative; `n > 0` guard catches this               |
| Negative powers of 2 | **Not** power of 2 | `n > 0` guard catches all negatives         |
| `INT_MIN`      | **Not** power of 2 | Despite having one bit set in [[Twos Complement]], it is negative |

```ad-warning
title: The Zero Trap
Without the `n > 0` check, `(0 & (0 - 1)) == 0` is true, falsely indicating zero is a power of two. Always include the positive check.
```

---

## Alternative: Using Lowest Set Bit Isolation

**Formula:** `n > 0 && (n & -n) == n`

The expression `n & -n` isolates the [[Isolate and Clear Lowest Set Bit|lowest set bit]]. If that isolated bit equals `n` itself, then `n` has only one bit set.

```
n = 8:
  n    = 00001000
  -n   = 11111000    (two's complement)
  n&-n = 00001000    == n  --> Power of 2!

n = 6:
  n    = 00000110
  -n   = 11111010    (two's complement)
  n&-n = 00000010    != n  --> NOT a power of 2
```

This works because in [[Twos Complement]], `-n = ~n + 1`. See [[Isolate and Clear Lowest Set Bit]] for the full explanation.

---

## Built-in Alternatives

Modern languages provide optimized intrinsics:

| Language   | Function                        | Notes                                     |
| ---------- | ------------------------------- | ----------------------------------------- |
| C#         | `BitOperations.IsPow2(n)`      | .NET 6+ / `System.Numerics`              |
| C++        | `std::has_single_bit(n)`        | C++20 / `<bit>` header, unsigned types    |
| JavaScript | (none)                          | Use the manual `n > 0 && (n & (n-1)) === 0` |

```ad-tip
title: Prefer Built-ins When Available
Built-in functions like `BitOperations.IsPow2` may compile down to specialized CPU instructions and are clearer to read. Use the bit trick when built-ins are unavailable or when you need to understand the mechanics.
```

---

## Code Examples

### C\#

```csharp
using System.Numerics;

public static class PowerOfTwo
{
    /// <summary>Check using the classic bit trick.</summary>
    public static bool IsPowerOfTwo(int n)
    {
        return n > 0 && (n & (n - 1)) == 0;
    }

    /// <summary>Check using lowest-set-bit isolation.</summary>
    public static bool IsPowerOfTwoAlt(int n)
    {
        return n > 0 && (n & -n) == n;
    }

    /// <summary>Check using built-in (.NET 6+).</summary>
    public static bool IsPowerOfTwoBuiltin(int n)
    {
        // BitOperations.IsPow2 works on unsigned types,
        // so guard against negatives first
        return n > 0 && BitOperations.IsPow2((uint)n);
    }
}

// Test
int[] values = { 0, 1, 2, 3, 4, 5, 8, 16, 15, 64, 100, 128 };
foreach (int v in values)
{
    Console.WriteLine($"{v,4}: {PowerOfTwo.IsPowerOfTwo(v)}");
}
// Output:
//    0: False
//    1: True
//    2: True
//    3: False
//    4: True
//    5: False
//    8: True
//   16: True
//   15: False
//   64: True
//  100: False
//  128: True
```

### C++

```cpp
#include <iostream>
#include <bit>       // C++20 for std::has_single_bit

// Classic bit trick
bool isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}

// Lowest-set-bit isolation
bool isPowerOfTwoAlt(int n) {
    return n > 0 && (n & -n) == n;
}

// C++20 built-in (unsigned types only)
bool isPowerOfTwoBuiltin(unsigned int n) {
    return n > 0 && std::has_single_bit(n);
}

int main() {
    int values[] = {0, 1, 2, 3, 4, 5, 8, 16, 15, 64, 100, 128};

    for (int v : values) {
        std::cout << v << ": "
                  << (isPowerOfTwo(v) ? "true" : "false")
                  << "\n";
    }

    return 0;
}
```

### JavaScript

```javascript
// Classic bit trick
function isPowerOfTwo(n) {
    return n > 0 && (n & (n - 1)) === 0;
}

// Lowest-set-bit isolation
function isPowerOfTwoAlt(n) {
    return n > 0 && (n & -n) === n;
}

// Test
const values = [0, 1, 2, 3, 4, 5, 8, 16, 15, 64, 100, 128];
values.forEach(v => {
    console.log(`${v}: ${isPowerOfTwo(v)}`);
});
// Output:
// 0: false
// 1: true
// 2: true
// 3: false
// 4: true
// ...
```

```ad-note
title: JavaScript Limitation
JavaScript bitwise operators work on 32-bit signed integers. For values beyond 2^31 - 1, use BigInt: `(n > 0n && (n & (n - 1n)) === 0n)`.
```

---

## Related Concepts

- **[[Count Set Bits]]** -- if `popcount(n) == 1`, then `n` is a power of two. This is a more expensive but conceptually clear alternative.
- **[[Isolate and Clear Lowest Set Bit]]** -- `n & (n - 1)` clears the lowest set bit; `n & -n` isolates it. Both are used in the power-of-two check.
- **[[AND Operator]]** -- the operator that makes the comparison work.
- **[[Binary Number System]]** -- understanding binary representation is essential.
- **[[Twos Complement]]** -- explains why `-n` works the way it does in the alternative formula.

---

## Summary

| Approach              | Formula                                 | Complexity |
| --------------------- | --------------------------------------- | ---------- |
| Classic bit trick     | `n > 0 && (n & (n - 1)) == 0`         | O(1)       |
| Lowest set bit        | `n > 0 && (n & -n) == n`              | O(1)       |
| Popcount              | `n > 0 && popcount(n) == 1`           | O(1)*      |
| Built-in (C#)         | `BitOperations.IsPow2(n)`             | O(1)       |
| Built-in (C++20)      | `std::has_single_bit(n)`               | O(1)       |

\* Popcount is O(1) when using hardware instructions, O(log n) otherwise.
