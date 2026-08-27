---
tags:
  - bit-manipulation
  - technique
  - bit-operations
---

## Check Set and Clear Bits

The most fundamental bit manipulation operations are **checking**, **setting**, and **clearing** individual bits within an integer. These three operations form the backbone of virtually every bit manipulation technique and are essential building blocks for working with [[Flag Enums and Bit Flags]], [[Creating Bit Masks]], and low-level systems programming.

Every operation below relies on creating a single-bit mask using the [[Left Shift]] operator and then combining it with [[AND Operator]], [[OR Operator]], or [[NOT Operator]].

---

## Check If Bit N Is Set

To test whether bit N is `1`, create a mask with only bit N set, then [[AND Operator|AND]] it with the value. If the result is non-zero, the bit is set.

**Formula:** `(value & (1 << n)) != 0`

**Step-by-step walkthrough** (checking bit 3 of value `0b11010110`, which is 214):

```
Step 1: Create the mask with (1 << 3)

    00000001      (the number 1)
    << 3          (shift left by 3)
  = 00001000      (mask with only bit 3 set)

Step 2: AND the value with the mask

    11010110      (value = 214)
  & 00001000      (mask)
  ----------
    00000000      (result = 0, so bit 3 is NOT set)
```

Now checking bit 4 of the same value:

```
Step 1: Create the mask with (1 << 4)

    00000001      (the number 1)
    << 4          (shift left by 4)
  = 00010000      (mask with only bit 4 set)

Step 2: AND the value with the mask

    11010110      (value = 214)
  & 00010000      (mask)
  ----------
    00010000      (result != 0, so bit 4 IS set)
```

```ad-tip
title: Why != 0 instead of == 1?
The AND result preserves the bit in its original position. If you check bit 4, the result is `16` (not `1`). Always compare to zero rather than to one, or right-shift the result if you need a boolean `0`/`1`.
```

---

## Set Bit N (Turn It On)

To force bit N to `1` regardless of its current state, [[OR Operator|OR]] the value with a mask that has only bit N set.

**Formula:** `value | (1 << n)`

Recall the [[OR Operator]] truth table:

```
  A | B | A OR B
  --|---|-------
  0 | 0 |   0
  0 | 1 |   1      <-- mask bit is 1, forces result to 1
  1 | 0 |   1      <-- original bit preserved
  1 | 1 |   1
```

The mask has `0` in every position except bit N. OR with `0` preserves the original bit; OR with `1` forces it on.

```
Example: Set bit 0 of value 11010110

    11010110      (value = 214)
  | 00000001      (mask = 1 << 0)
  ----------
    11010111      (result = 215, bit 0 is now set)
```

---

## Clear Bit N (Turn It Off)

To force bit N to `0`, [[AND Operator|AND]] the value with an inverted mask: a mask that has `1` everywhere **except** bit N.

**Formula:** `value & ~(1 << n)`

```
Step 1: Create the mask

    00000001      (1)
    << 4          (shift left by 4)
  = 00010000      (single-bit mask)

Step 2: Invert the mask with NOT

  ~ 00010000
  = 11101111      (all 1s except bit 4)

Step 3: AND with the value

    11010110      (value = 214)
  & 11101111      (inverted mask)
  ----------
    11000110      (result = 198, bit 4 is now cleared)
```

The [[NOT Operator]] inverts every bit, so the mask becomes a "hole" at position N. AND with `1` preserves; AND with `0` clears.

---

## Modify Bit N to a Given Value

Sometimes you need to set a bit to a specific value (0 or 1) that comes from a variable or condition. This combines clearing and setting in one expression.

**Formula:** `(value & ~(1 << n)) | (bit << n)`

```
Step 1: Clear bit N            -> value & ~(1 << n)
Step 2: Shift the new bit      -> bit << n
Step 3: OR them together       -> combines the cleared value with the new bit

Example: Set bit 2 of 11010110 to 1

    11010110            (original value)
  & 11111011            (~(1 << 2) = inverted mask)
  ----------
    11010010            (bit 2 cleared)
  | 00000100            (1 << 2 = new bit in position)
  ----------
    11010110            (bit 2 set to 1)
```

```ad-note
title: Idempotent Operation
This formula works correctly whether the target bit was already 0 or 1. It always clears first, then sets, avoiding any toggle artifacts.
```

---

## Code Examples

### C\#

```csharp
public static class BitOps
{
    /// <summary>Check if bit n is set (returns true/false).</summary>
    public static bool IsBitSet(int value, int n)
    {
        return (value & (1 << n)) != 0;
    }

    /// <summary>Set bit n to 1.</summary>
    public static int SetBit(int value, int n)
    {
        return value | (1 << n);
    }

    /// <summary>Clear bit n to 0.</summary>
    public static int ClearBit(int value, int n)
    {
        return value & ~(1 << n);
    }

    /// <summary>Set bit n to a specific value (0 or 1).</summary>
    public static int ModifyBit(int value, int n, int bit)
    {
        return (value & ~(1 << n)) | (bit << n);
    }
}

// Usage
int flags = 0b_1101_0110;  // 214

Console.WriteLine(BitOps.IsBitSet(flags, 4));   // True
Console.WriteLine(BitOps.IsBitSet(flags, 3));   // False

flags = BitOps.SetBit(flags, 0);                // 0b_1101_0111 = 215
flags = BitOps.ClearBit(flags, 7);              // 0b_0101_0111 = 87
flags = BitOps.ModifyBit(flags, 2, 0);          // 0b_0101_0011 = 83
```

### C++

```cpp
#include <iostream>
#include <cstdint>

// Check if bit n is set
bool isBitSet(int value, int n) {
    return (value & (1 << n)) != 0;
}

// Set bit n to 1
int setBit(int value, int n) {
    return value | (1 << n);
}

// Clear bit n to 0
int clearBit(int value, int n) {
    return value & ~(1 << n);
}

// Set bit n to a specific value (0 or 1)
int modifyBit(int value, int n, int bit) {
    return (value & ~(1 << n)) | (bit << n);
}

int main() {
    int flags = 0b11010110;  // 214

    std::cout << std::boolalpha;
    std::cout << "Bit 4 set? " << isBitSet(flags, 4) << "\n";   // true
    std::cout << "Bit 3 set? " << isBitSet(flags, 3) << "\n";   // false

    flags = setBit(flags, 0);       // 0b11010111 = 215
    flags = clearBit(flags, 7);     // 0b01010111 = 87
    flags = modifyBit(flags, 2, 0); // 0b01010011 = 83

    std::cout << "Result: " << flags << "\n";
    return 0;
}
```

### JavaScript

```javascript
// Check if bit n is set
function isBitSet(value, n) {
    return (value & (1 << n)) !== 0;
}

// Set bit n to 1
function setBit(value, n) {
    return value | (1 << n);
}

// Clear bit n to 0
function clearBit(value, n) {
    return value & ~(1 << n);
}

// Set bit n to a specific value (0 or 1)
function modifyBit(value, n, bit) {
    return (value & ~(1 << n)) | (bit << n);
}

// Usage
let flags = 0b11010110;  // 214

console.log(isBitSet(flags, 4));   // true
console.log(isBitSet(flags, 3));   // false

flags = setBit(flags, 0);          // 0b11010111 = 215
flags = clearBit(flags, 7);        // 0b01010111 = 87
flags = modifyBit(flags, 2, 0);    // 0b01010011 = 83

console.log(flags);                // 83
```

```ad-warning
title: Shift Amount Limits
In C# and JavaScript, shifting by more than the bit width of the type is undefined or wraps around. In C#, `1 << 32` on an `int` wraps to `1 << 0`. In JavaScript, bitwise ops work on 32-bit integers, so bit indices 0-31 are valid. In C++, shifting by >= the bit width is **undefined behavior**.
```

---

## Practical Example: Boolean Flags in a Single Integer

Instead of using 8 separate `bool` variables, pack them into a single byte. This is exactly what [[Flag Enums and Bit Flags]] formalizes.

```csharp
// Define flag positions
const int FLAG_VISIBLE   = 0;
const int FLAG_ENABLED   = 1;
const int FLAG_SELECTED  = 2;
const int FLAG_FOCUSED   = 3;
const int FLAG_READONLY  = 4;
const int FLAG_DIRTY     = 5;

int state = 0;

// Set the widget to visible and enabled
state = BitOps.SetBit(state, FLAG_VISIBLE);    // state = 0b000001
state = BitOps.SetBit(state, FLAG_ENABLED);    // state = 0b000011

// Check if visible
if (BitOps.IsBitSet(state, FLAG_VISIBLE))
    Console.WriteLine("Widget is visible");

// User selects the widget
state = BitOps.SetBit(state, FLAG_SELECTED);   // state = 0b000111

// Disable the widget
state = BitOps.ClearBit(state, FLAG_ENABLED);  // state = 0b000101

// Toggle read-only based on a condition
bool makeReadOnly = true;
state = BitOps.ModifyBit(state, FLAG_READONLY,
    makeReadOnly ? 1 : 0);                     // state = 0b010101
```

```
Bit layout of state:

  Bit:  7  6  5  4  3  2  1  0
        |  |  |  |  |  |  |  |
        0  0  0  1  0  1  0  1
              |  |  |  |  |  └── VISIBLE   (set)
              |  |  |  |  └──── ENABLED   (cleared)
              |  |  |  └────── SELECTED  (set)
              |  |  └──────── FOCUSED   (cleared)
              |  └────────── READONLY  (set)
              └──────────── DIRTY     (cleared)
```

---

## Summary

| Operation          | Formula                              | Key Operator              |
| ------------------ | ------------------------------------ | ------------------------- |
| Check bit N        | `(value & (1 << n)) != 0`           | [[AND Operator]]          |
| Set bit N          | `value \| (1 << n)`                 | [[OR Operator]]           |
| Clear bit N        | `value & ~(1 << n)`                 | [[NOT Operator]] + AND    |
| Modify bit N       | `(value & ~(1 << n)) \| (bit << n)` | Clear + Set               |

These operations are the foundation for [[Toggle Bits]], [[Flag Enums and Bit Flags]], and [[Creating Bit Masks]].
