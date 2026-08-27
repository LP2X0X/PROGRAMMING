---
title: NOT Operator (Bitwise Complement)
date: 2026-08-19
tags:
  - bit-manipulation
  - operator
  - bitwise-not
aliases:
  - Bitwise Complement
  - Bitwise NOT
  - Ones Complement Operator
---

## NOT Operator (Bitwise Complement)

The **NOT operator** (`~`) is the only **unary** bitwise operator — it takes a single operand and **flips every bit**. Every `0` becomes `1`, and every `1` becomes `0`. This operation is also called the **bitwise complement** or **ones' complement**.

Unlike [[AND Operator]], [[OR Operator]], and [[XOR Operator]], which combine two values, NOT works on a single value in isolation.

---

## Truth Table

The NOT operation on a single bit is the simplest possible truth table:

| Input | Output (`~Input`) |
|:-----:|:-----------------:|
|   0   |         1         |
|   1   |         0         |

When applied to a multi-bit value, NOT flips **every bit independently**. Here is an example on a full 8-bit value:

```
Original:    0 1 1 0  1 0 1 0    (0x6A = 106)
             ~ ~ ~ ~  ~ ~ ~ ~
Result:      1 0 0 1  0 1 0 1    (0x95 = 149 unsigned, -107 signed)
```

Every single bit position is inverted. There are no carries, no interaction between bit positions — each bit is processed on its own.

---

## ASCII Diagram — NOT on an 8-bit Value

```
  Bit Position:   7   6   5   4   3   2   1   0
                ┌───┬───┬───┬───┬───┬───┬───┬───┐
  Input (0x6A): │ 0 │ 1 │ 1 │ 0 │ 1 │ 0 │ 1 │ 0 │  = 106
                └─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┴─┬─┘
                  │   │   │   │   │   │   │   │
                  ▼   ▼   ▼   ▼   ▼   ▼   ▼   ▼     FLIP each bit
                ┌───┬───┬───┬───┬───┬───┬───┬───┐
  Output(0x95): │ 1 │ 0 │ 0 │ 1 │ 0 │ 1 │ 0 │ 1 │  = 149 (unsigned)
                └───┴───┴───┴───┴───┴───┴───┴───┘    = -107 (signed)
```

---

## Syntax Across Languages

The NOT operator uses the tilde (`~`) symbol in C#, C++, and JavaScript:

| Language   | Syntax  | Operand Type                | Result Type           |
|:-----------|:--------|:----------------------------|:----------------------|
| C#         | `~x`    | `int`, `uint`, `long`, etc. | Same as operand       |
| C++        | `~x`    | Any integer type            | Promoted integer type |
| JavaScript | `~x`    | Any value                   | 32-bit signed integer |

```ad-warning
title: Do Not Confuse ~ with !
The `~` operator is the **bitwise** NOT — it flips every bit in the binary representation. The `!` operator is the **logical** NOT — it flips a boolean value between `true` and `false`. They are completely different operations.

- `~5` flips all bits of 5 -> produces `-6`
- `!true` flips boolean -> produces `false`
```

---

## Relationship with Two's Complement

On any system that uses [[Twos Complement]] representation (which is virtually every modern system), the NOT operator has a clean mathematical identity:

```
~x = -x - 1
```

Or equivalently:

```
~x = -(x + 1)
```

**Why does this hold?** In [[Twos Complement]], the negative of a number is computed by flipping all bits and adding 1:

```
-x = ~x + 1
```

Rearranging that equation:

```
~x = -x - 1
```

Here is a concrete walkthrough with an 8-bit signed value:

```
x = 5:       00000101

Step 1 — Flip all bits (~x):
~5:          11111010    = -6 in two's complement

Step 2 — Verify: -x - 1 = -5 - 1 = -6  ✓
```

Some practical consequences of this identity:

| Expression | Value | Because          |
|:-----------|:------|:-----------------|
| `~0`       | `-1`  | `-(0) - 1 = -1` |
| `~1`       | `-2`  | `-(1) - 1 = -2` |
| `~(-1)`    | `0`   | `-(-1) - 1 = 0` |
| `~(-5)`    | `4`   | `-(-5) - 1 = 4` |

```ad-note
title: Why ~0 Produces All Ones
`~0` flips every bit from 0 to 1, producing a value where every bit is set. In [[Twos Complement]], a value with all bits set is `-1` regardless of the integer width (8-bit, 16-bit, 32-bit, 64-bit). This makes `~0` a portable way to get "all ones."
```

---

## Creating Inverse Masks Using NOT

One of the most common uses of the NOT operator is creating **inverse masks** for [[Creating Bit Masks|bit masking]] operations. If you have a mask that selects certain bits, `~mask` gives you the complementary mask that selects everything else.

**Use case — clearing specific bits:**

To clear bits 3 and 4 of a value, you first define a mask with those bits set, then NOT it and AND:

```
mask:           00011000    (bits 3 and 4 set)
~mask:          11100111    (bits 3 and 4 cleared, everything else set)
value & ~mask:  clears bits 3 and 4, preserves all others
```

This is the standard [[Check Set and Clear Bits|clear bits]] pattern:

```
value = value & ~mask;    // Clear the bits specified by mask
```

```ad-tip
title: Prefer ~mask Over Hardcoding the Inverse
Writing `value & ~0x18` is better than `value & 0xE7` because the intent is clearer — you are saying "clear these bits" rather than making the reader figure out which bits 0xE7 preserves. It also avoids errors if the integer width changes.
```

**Use case — toggling between two sets of flags:**

If you maintain an "enabled" mask and a "disabled" mask, one is always the NOT of the other:

```
uint enabledFlags = 0x0F;         // Lower nibble enabled
uint disabledFlags = ~enabledFlags; // Upper bits enabled, lower disabled
```

---

## Gotcha: Result Depends on Integer Width

The result of NOT depends entirely on the **width** of the integer type. The same logical value produces different results at different widths because there are more or fewer bits to flip.

```
8-bit  ~0x0F:   ~00001111                     = 11110000         = 0xF0
16-bit ~0x0F:   ~00000000 00001111            = 11111111 11110000 = 0xFFF0
32-bit ~0x0F:   ~00000000 ... 00001111        = 11111111 ... 11110000 = 0xFFFFFFF0
```

```ad-warning
title: Width Matters for Masks
If you create a mask with `~0x0F` expecting `0xF0`, you will get `0xFFFFFFF0` in a 32-bit context. This can cause incorrect behavior when applied to 8-bit data unless you also mask to the desired width:

```cpp
uint8_t byte_mask = (uint8_t)(~0x0F);  // Cast to truncate to 8 bits = 0xF0
uint32_t wide_mask = ~0x0F;            // All upper bits set = 0xFFFFFFF0
```
```

This is closely related to how [[Signed and Unsigned Integers]] handle their bit patterns at different widths, and how [[Bits Bytes and Words]] defines the fundamental storage units.

---

## JavaScript-Specific Gotcha

In JavaScript, all numbers are IEEE 754 64-bit floating point. When you apply `~`, JavaScript:

1. Converts the operand to a **32-bit signed integer** (truncating any fractional part)
2. Flips all 32 bits
3. Returns the result as a standard JavaScript number

This means:

```javascript
// Floating point is truncated before NOT
~4.9    // same as ~4 -> -5

// Values beyond 32-bit range wrap around
~(2**32)  // 2^32 wraps to 0 in 32-bit, then ~0 = -1

// The double-tilde idiom truncates to 32-bit integer
~~4.9   // 4  (truncates decimal, like Math.trunc for 32-bit range)
```

```ad-warning
title: Double Tilde (~~) is NOT Math.trunc
The `~~value` trick is sometimes used to truncate a float to an integer. However, it only works correctly for values within the 32-bit signed integer range (-2,147,483,648 to 2,147,483,647). Values outside this range will give incorrect results. Prefer `Math.trunc()` for clarity and correctness.
```

The `~` operator in JavaScript also has a legacy idiom with `indexOf`:

```javascript
// Old pattern (before .includes() existed):
if (~str.indexOf("search")) {
    // found — because indexOf returns -1 for not found, and ~(-1) = 0 (falsy)
}

// Modern equivalent — prefer this:
if (str.includes("search")) {
    // found
}
```

This works because `~(-1) = 0` (falsy) and `~(any other index) != 0` (truthy), effectively converting the `-1` sentinel to a boolean-like check.

---

## Full Code Examples

### C++ Example

```cpp
#include <cstdint>
#include <cstdio>
#include <bitset>
#include <iostream>

int main() {
    // Basic NOT operation
    uint8_t value = 0b01101010;  // 0x6A = 106
    uint8_t flipped = ~value;    // 0x95 = 149

    printf("Original:  0x%02X (%d)\n", value, value);
    printf("Flipped:   0x%02X (%d)\n", flipped, flipped);

    // Two's complement identity: ~x = -x - 1
    int x = 42;
    printf("~%d = %d (expected %d)\n", x, ~x, -x - 1);  // ~42 = -43

    // Creating an inverse mask to clear bits
    uint8_t data = 0b11111111;
    uint8_t mask = 0b00011000;     // Bits 3 and 4
    uint8_t cleared = data & ~mask; // Clear bits 3 and 4
    // Result: 0b11100111

    std::cout << "Data:    " << std::bitset<8>(data) << std::endl;
    std::cout << "Mask:    " << std::bitset<8>(mask) << std::endl;
    std::cout << "~Mask:   " << std::bitset<8>((uint8_t)~mask) << std::endl;
    std::cout << "Cleared: " << std::bitset<8>(cleared) << std::endl;

    // Width demonstration
    uint8_t  not8  = ~(uint8_t)0x0F;   // 0xF0
    uint16_t not16 = ~(uint16_t)0x0F;  // 0xFFF0
    uint32_t not32 = ~(uint32_t)0x0F;  // 0xFFFFFFF0

    printf("\n~0x0F at different widths:\n");
    printf("  8-bit:  0x%02X\n", not8);
    printf(" 16-bit:  0x%04X\n", not16);
    printf(" 32-bit:  0x%08X\n", not32);

    return 0;
}
```

### C# Example

```csharp
using System;

class BitwiseNotDemo
{
    static void Main()
    {
        // Basic NOT operation
        byte value = 0b_0110_1010;  // 0x6A = 106
        // Note: ~ on a byte promotes to int in C#, cast back
        byte flipped = (byte)~value; // 0x95 = 149

        Console.WriteLine($"Original: 0x{value:X2} ({value})");
        Console.WriteLine($"Flipped:  0x{flipped:X2} ({flipped})");

        // Two's complement identity: ~x = -x - 1
        int x = 42;
        Console.WriteLine($"~{x} = {~x} (expected {-x - 1})");  // ~42 = -43

        // Creating an inverse mask to clear bits
        byte data = 0b_1111_1111;
        byte mask = 0b_0001_1000;           // Bits 3 and 4
        byte cleared = (byte)(data & ~mask); // Clear bits 3 and 4

        Console.WriteLine($"\nData:    {Convert.ToString(data, 2).PadLeft(8, '0')}");
        Console.WriteLine($"Mask:    {Convert.ToString(mask, 2).PadLeft(8, '0')}");
        Console.WriteLine($"Cleared: {Convert.ToString(cleared, 2).PadLeft(8, '0')}");

        // ~0 gives all ones (-1 in signed)
        int allOnes = ~0;
        Console.WriteLine($"\n~0 = {allOnes} (0x{allOnes:X8})");

        // Using NOT with uint for unsigned operations
        uint flags = 0x0000_00FF;
        uint inverted = ~flags;  // 0xFFFFFF00
        Console.WriteLine($"\n~0x{flags:X8} = 0x{inverted:X8}");

        // Practical: clear bit 5 of a register value
        int register_val = 0xFF;
        int bit5Mask = 1 << 5;                        // 0x20 = 0b00100000
        int result = register_val & ~bit5Mask;         // Clears bit 5
        Console.WriteLine($"\nClear bit 5 of 0x{register_val:X2}: 0x{result:X2}");
    }
}
```

### JavaScript Example

```javascript
// Basic NOT operation
let value = 0b01101010;  // 0x6A = 106
let flipped = ~value;    // Operates on 32-bit signed integer

console.log(`Original: ${value} (0b${value.toString(2).padStart(8, '0')})`);
console.log(`~value:   ${flipped}`);  // -107 (32-bit result, not 8-bit)

// Two's complement identity: ~x = -x - 1
let x = 42;
console.log(`~${x} = ${~x} (expected ${-x - 1})`);  // ~42 = -43

// JavaScript always uses 32-bit signed integers for ~
console.log(`~0 = ${~0}`);        // -1
console.log(`~(-1) = ${~(-1)}`);  //  0

// The indexOf idiom (legacy — prefer .includes())
let str = "hello world";
if (~str.indexOf("world")) {
    console.log("Found 'world'");
}
// Modern equivalent:
if (str.includes("world")) {
    console.log("Found 'world' (modern)");
}

// Double-tilde truncation (32-bit range only!)
console.log(`~~4.9 = ${~~4.9}`);           // 4
console.log(`~~(-3.7) = ${~~(-3.7)}`);     // -3
console.log(`~~(2**31 - 1) = ${~~(2**31 - 1)}`);  // 2147483647 (max safe)
console.log(`~~(2**31) = ${~~(2**31)}`);           // -2147483648 (overflow!)

// Creating a mask and its inverse
let mask = 0x0F;           // Lower nibble
let inverseMask = ~mask;   // 0xFFFFFFF0 (32-bit)
// To get an 8-bit result, mask it:
let inverseMask8 = (~mask) & 0xFF;  // 0xF0

console.log(`\nmask:       0x${(mask >>> 0).toString(16).toUpperCase()}`);
console.log(`~mask:      0x${(inverseMask >>> 0).toString(16).toUpperCase()}`);
console.log(`~mask & FF: 0x${inverseMask8.toString(16).toUpperCase()}`);
```

---

## Common Patterns and Idioms

| Pattern                   | Code              | Purpose                                               |
|:--------------------------|:------------------|:------------------------------------------------------|
| Clear specific bits       | `x & ~mask`       | Turn off bits where mask is 1 — see [[Check Set and Clear Bits]] |
| Get all-ones value        | `~0`              | Portable way to fill all bits with 1                  |
| Negate via complement     | `~x + 1`          | Equivalent to `-x` in [[Twos Complement]]             |
| Check if -1 (not found)   | `~indexOf()`      | Legacy JS idiom, falsy when indexOf returns -1        |
| Truncate to integer       | `~~floatVal`      | JS only, 32-bit range — prefer `Math.trunc()`        |
| Invert a flag set         | `~flags`          | Get complement of a set of [[Creating Bit Masks|bit flags]] |

---

## Key Takeaways

- The NOT operator (`~`) flips every bit in a value — it is the only unary bitwise operator
- In [[Twos Complement]] systems, `~x` is mathematically equal to `-x - 1`
- `~0` always produces all ones (`-1` in signed representation), regardless of integer width
- The result of NOT depends on the **integer width** — always be aware of whether you are working with 8, 16, 32, or 64 bits
- The primary practical use is creating **inverse masks** for the [[Check Set and Clear Bits|clear bits]] pattern: `value & ~mask`
- In JavaScript, `~` always converts to 32-bit signed integer first, which can produce surprising results with large numbers or floats
- Do not confuse `~` (bitwise NOT) with `!` (logical NOT)

---

## Related Topics

- [[Binary Number System]] — the foundation for understanding what "flipping bits" means
- [[Bits Bytes and Words]] — why integer width affects the NOT result
- [[Twos Complement]] — the encoding that gives NOT its `-x - 1` identity
- [[Signed and Unsigned Integers]] — how the same bit pattern after NOT is interpreted differently
- [[Creating Bit Masks]] — NOT is essential for creating complementary masks
- [[Check Set and Clear Bits]] — the `value & ~mask` pattern uses NOT directly
- [[AND Operator]] — often combined with NOT for the clear-bits pattern
- [[OR Operator]] — the other combining operator for bit manipulation
- [[XOR Operator]] — another bit-flipping operator, but binary rather than unary
