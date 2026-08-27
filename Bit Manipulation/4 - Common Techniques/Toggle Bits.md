---
tags:
  - bit-manipulation
  - technique
  - xor
---

## Toggle Bits

Toggling a bit means flipping it from `0` to `1` or from `1` to `0`. The [[XOR Operator]] is the perfect tool for this because of its unique truth table property: XOR with `1` always inverts the bit, while XOR with `0` always preserves it.

---

## Why XOR Is the Toggle Operator

Recall the [[XOR Operator]] truth table:

```
  A | B | A XOR B
  --|---|--------
  0 | 0 |   0       <-- B=0: A is preserved
  0 | 1 |   1       <-- B=1: A is flipped
  1 | 0 |   1       <-- B=0: A is preserved
  1 | 1 |   0       <-- B=1: A is flipped
```

Key insight:
- **XOR with `0`** = keep the original bit (identity)
- **XOR with `1`** = flip the bit (toggle)

This means we can selectively toggle specific bits by [[Creating Bit Masks|creating a mask]] where `1` marks the bits we want to flip and `0` marks the bits we want to preserve.

---

## Toggle a Single Bit

**Formula:** `value ^ (1 << n)`

Create a mask with only bit N set using [[Left Shift]], then XOR.

```
Example: Toggle bit 2 of 11010110 (214)

Step 1: Create mask
    00000001      (1)
    << 2          (shift left by 2)
  = 00000100      (mask with only bit 2 set)

Step 2: XOR with value
    11010110      (value = 214)
  ^ 00000100      (mask)
  ----------
    11010010      (result = 210, bit 2 flipped from 1 to 0)
```

Now toggle bit 3 (which is currently 0):

```
    11010010      (value = 210)
  ^ 00001000      (1 << 3)
  ----------
    11011010      (result = 218, bit 3 flipped from 0 to 1)
```

---

## Toggle Multiple Bits

**Formula:** `value ^ mask`

To flip several bits at once, create a mask with `1` in every position you want to toggle.

```
Example: Toggle bits 0, 1, and 7 of 11010110 (214)

    Mask = (1 << 0) | (1 << 1) | (1 << 7)
         = 00000001 | 00000010 | 10000000
         = 10000011

    11010110      (value = 214)
  ^ 10000011      (mask)
  ----------
    01010101      (result = 85)

  Bit 0: was 0, flipped to 1
  Bit 1: was 1, flipped to 0
  Bit 7: was 1, flipped to 0
  All other bits: unchanged
```

---

## Toggling Is Its Own Inverse

One of the most powerful properties of XOR: applying the same toggle twice restores the original value.

**Property:** `(value ^ mask) ^ mask == value`

```
Proof with concrete values:

    Original:    11010110     (214)
  ^ Mask:        00001111
  = First XOR:   11011001     (217)
  ^ Mask:        00001111
  = Second XOR:  11010110     (214)  <-- back to original!
```

**Why this works mathematically:**

```
(value ^ mask) ^ mask
= value ^ (mask ^ mask)     (XOR is associative)
= value ^ 0                 (anything XOR itself = 0)
= value                     (anything XOR 0 = itself)
```

```ad-tip
title: Self-Inverse Property
This self-inverse property is the foundation of [[Swap Without Temp Variable|XOR swap]], simple encryption/obfuscation schemes, and [[Find the Lone Non-Duplicate|finding unique elements]].
```

---

## Practical Example: Toggling ASCII Letter Case

In the ASCII encoding, uppercase and lowercase letters differ by exactly **one bit** -- bit 5 (value 32):

```
'A' = 01000001  (65)
'a' = 01100001  (97)      difference = 32 = bit 5
      ----^---

'B' = 01000010  (66)
'b' = 01100010  (98)      difference = 32 = bit 5
      ----^---

'Z' = 01011010  (90)
'z' = 01111010  (122)     difference = 32 = bit 5
      ----^---
```

XOR with 32 (`0b00100000`) toggles bit 5, converting between upper and lowercase:

```
    01000001      ('A' = 65)
  ^ 00100000      (32 = 1 << 5)
  ----------
    01100001      ('a' = 97)

    01100001      ('a' = 97)
  ^ 00100000      (32 = 1 << 5)
  ----------
    01000001      ('A' = 65)
```

```ad-note
title: Case Conversion Shortcuts
Using the [[AND Operator]] and [[OR Operator]]:
- **Force uppercase:** `ch & ~32` (clear bit 5) -- equivalent to `ch & 0xDF`
- **Force lowercase:** `ch | 32` (set bit 5) -- equivalent to `ch | 0x20`
- **Toggle case:** `ch ^ 32` (toggle bit 5)

These only work for ASCII letters (A-Z, a-z). See [[Check Set and Clear Bits]] for the clear/set operations.
```

---

## Code Examples

### C\#

```csharp
public static class BitToggle
{
    /// <summary>Toggle bit n of a value.</summary>
    public static int ToggleBit(int value, int n)
    {
        return value ^ (1 << n);
    }

    /// <summary>Toggle multiple bits using a mask.</summary>
    public static int ToggleBits(int value, int mask)
    {
        return value ^ mask;
    }

    /// <summary>Toggle the case of an ASCII letter.</summary>
    public static char ToggleCase(char ch)
    {
        return (char)(ch ^ 32);
    }
}

// Usage
int flags = 0b_1101_0110;  // 214

// Toggle bit 2 (was 1, becomes 0)
flags = BitToggle.ToggleBit(flags, 2);   // 0b_1101_0010 = 210

// Toggle it back (was 0, becomes 1)
flags = BitToggle.ToggleBit(flags, 2);   // 0b_1101_0110 = 214

// Toggle multiple bits at once
int mask = (1 << 0) | (1 << 1) | (1 << 7);
flags = BitToggle.ToggleBits(flags, mask);  // 0b_0101_0101 = 85

// Toggle ASCII case
Console.WriteLine(BitToggle.ToggleCase('A'));  // 'a'
Console.WriteLine(BitToggle.ToggleCase('z'));  // 'Z'
```

### C++

```cpp
#include <iostream>
#include <cstdint>

// Toggle bit n of a value
int toggleBit(int value, int n) {
    return value ^ (1 << n);
}

// Toggle multiple bits using a mask
int toggleBits(int value, int mask) {
    return value ^ mask;
}

// Toggle the case of an ASCII letter
char toggleCase(char ch) {
    return static_cast<char>(ch ^ 32);
}

int main() {
    int flags = 0b11010110;  // 214

    // Toggle bit 2
    flags = toggleBit(flags, 2);  // 210
    std::cout << "After toggle bit 2: " << flags << "\n";

    // Toggle it back
    flags = toggleBit(flags, 2);  // 214
    std::cout << "After toggle again: " << flags << "\n";

    // Toggle multiple bits
    int mask = (1 << 0) | (1 << 1) | (1 << 7);
    flags = toggleBits(flags, mask);  // 85
    std::cout << "After multi-toggle: " << flags << "\n";

    // Toggle ASCII case
    std::cout << toggleCase('A') << "\n";  // 'a'
    std::cout << toggleCase('z') << "\n";  // 'Z'

    return 0;
}
```

### JavaScript

```javascript
// Toggle bit n of a value
function toggleBit(value, n) {
    return value ^ (1 << n);
}

// Toggle multiple bits using a mask
function toggleBits(value, mask) {
    return value ^ mask;
}

// Toggle the case of an ASCII letter
function toggleCase(ch) {
    return String.fromCharCode(ch.charCodeAt(0) ^ 32);
}

// Usage
let flags = 0b11010110;  // 214

// Toggle bit 2
flags = toggleBit(flags, 2);   // 210
console.log(`After toggle bit 2: ${flags}`);

// Toggle it back
flags = toggleBit(flags, 2);   // 214
console.log(`After toggle again: ${flags}`);

// Toggle multiple bits
const mask = (1 << 0) | (1 << 1) | (1 << 7);
flags = toggleBits(flags, mask);  // 85
console.log(`After multi-toggle: ${flags}`);

// Toggle ASCII case
console.log(toggleCase('A'));  // 'a'
console.log(toggleCase('z'));  // 'Z'
```

---

## Toggle vs Set/Clear

| Goal                     | Operation                   | Operator        |
| ------------------------ | --------------------------- | --------------- |
| Force bit to 1 (set)     | `value \| (1 << n)`        | [[OR Operator]] |
| Force bit to 0 (clear)   | `value & ~(1 << n)`        | [[AND Operator]] + [[NOT Operator]] |
| Flip bit (toggle)        | `value ^ (1 << n)`         | [[XOR Operator]] |

See [[Check Set and Clear Bits]] for the set/clear operations and [[Flag Enums and Bit Flags]] for how these operations are used with enum flags in practice.

```ad-warning
title: Toggle vs Set -- Know Which You Need
Toggling is dangerous when you do not know the current state of the bit. If you want a bit to be ON, use **set** (`|=`). If you use **toggle** (`^=`) and the bit is already on, you will accidentally turn it off. Only use toggle when you explicitly want the flip-to-opposite behavior.
```

---

## Summary

- XOR with `1` flips a bit; XOR with `0` preserves it.
- `value ^ (1 << n)` toggles a single bit at position N.
- `value ^ mask` toggles all bits where the mask has `1`.
- Toggling is self-inverse: applying it twice restores the original value.
- The ASCII case toggle trick (`ch ^ 32`) is a classic application.
- Related: [[XOR Operator]], [[Check Set and Clear Bits]], [[Creating Bit Masks]], [[Flag Enums and Bit Flags]].
