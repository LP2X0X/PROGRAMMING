---
tags:
  - bit-manipulation
  - bit-mask
  - fundamental
---

## What Is a Bit Mask?

A **bit mask** is a pattern of bits used to **select**, **set**, **clear**, or **toggle** specific bits within another value. The mask itself is just an integer, but its purpose is to act as a filter — it defines *which* bits an operation should affect and which it should leave alone.

Masks work hand-in-hand with the bitwise operators:

| Operation         | Operator             | What It Does with a Mask                           |
|-------------------|----------------------|----------------------------------------------------|
| Check if bit set  | [[AND Operator]] `&` | Isolate specific bits to test their state           |
| Set bits          | [[OR Operator]] `\|` | Force specific bits to `1`                         |
| Clear bits        | [[AND Operator]] `&` + [[NOT Operator]] `~` | Force specific bits to `0`   |
| Toggle bits       | [[XOR Operator]] `^` | Flip specific bits                                 |
| Invert mask       | [[NOT Operator]] `~` | Get the complement of a mask                       |

The key insight is that the mask never changes the target value by itself — it is always *combined* with a bitwise operator to produce a result. How you construct the mask determines which bits are targeted.

```ad-tip
title: Think of masks like stencils
A physical stencil lets paint through in some places and blocks it in others. A bit mask does the same thing: bits set to `1` in the mask mark the positions that "pass through" (are affected by the operation), while bits set to `0` are "blocked" (left unchanged).
```

---

## Single-Bit Masks

The most basic mask targets exactly one bit. The expression `1 << n` creates a mask with **only bit n** set to `1` and all other bits set to `0`.

### How `1 << n` Works

The value `1` in binary is `00000001` — a single `1` in the least significant position (bit 0). The [[Left Shift]] operator `<<` moves that `1` to the left by `n` positions, filling vacated positions with zeros.

```
  1         = 00000001   (bit 0 set)
  1 << 1    = 00000010   (bit 1 set)
  1 << 2    = 00000100   (bit 2 set)
  1 << 3    = 00001000   (bit 3 set)
  1 << 4    = 00010000   (bit 4 set)
  1 << 5    = 00100000   (bit 5 set)
  1 << 6    = 01000000   (bit 6 set)
  1 << 7    = 10000000   (bit 7 set)
```

### Step-by-Step: Creating a Mask for Bit 3

```
  Start:    00000001     decimal 1

  Shift 1:  00000010     the '1' moves left one position
  Shift 2:  00000100     moves again
  Shift 3:  00001000     three shifts total  <-- mask for bit 3

  Result:   1 << 3 = 0x08 = 8 in decimal
```

Each `1 << n` mask also equals `2^n` in decimal. This is no coincidence — shifting left by one is the same as multiplying by 2.

| Expression | Binary     | Decimal | Hex    |
|------------|------------|---------|--------|
| `1 << 0`   | `00000001` | 1       | `0x01` |
| `1 << 1`   | `00000010` | 2       | `0x02` |
| `1 << 2`   | `00000100` | 4       | `0x04` |
| `1 << 3`   | `00001000` | 8       | `0x08` |
| `1 << 4`   | `00010000` | 16      | `0x10` |
| `1 << 5`   | `00100000` | 32      | `0x20` |
| `1 << 6`   | `01000000` | 64      | `0x40` |
| `1 << 7`   | `10000000` | 128     | `0x80` |

```ad-warning
title: Bit numbering starts at 0
Bit positions are **zero-indexed**. Bit 0 is the rightmost (least significant) bit. A common off-by-one mistake is writing `1 << 3` expecting it to set the third bit from the right — it actually sets the **fourth** bit (position 3, counting from 0). If you want the bit at position 3, `1 << 3` is correct. If you want "the third bit," that is position 2, so use `1 << 2`.
```

---

## Multi-Bit Masks

To create a mask that targets **more than one bit**, combine multiple single-bit masks using the [[OR Operator]] `|`. Each `|` operation "adds" another `1` into the mask without disturbing the bits already set.

### Combining Two Bits

```
  (1 << 2)            = 00000100   bit 2 set
  (1 << 5)            = 00100000   bit 5 set

  (1 << 2) | (1 << 5) = 00100100   bits 2 and 5 set
                          ^  ^
                          |  |
                          |  +-- bit 2
                          +----- bit 5
```

### Combining Multiple Bits

You can chain as many `|` operations as needed:

```
  (1 << 0) | (1 << 3) | (1 << 7) = 10001001
                                     ^   ^  ^
                                     |   |  |
                                     |   |  +-- bit 0
                                     |   +----- bit 3
                                     +--------- bit 7
```

```ad-tip
title: Readability trick
When defining multi-bit masks in code, put each bit on its own line with a comment. This makes it easy to see which bits are included and why:
```c
const int mask = (1 << 0)   // Read permission
               | (1 << 1)   // Write permission
               | (1 << 2);  // Execute permission
```
```

---

## Range Masks

A **range mask** sets a contiguous block of bits from position `m` (low) through position `n` (high), inclusive. This is useful when you need to extract or manipulate a multi-bit field within a larger value.

### The Formula

```
rangeMask = ((1 << (n - m + 1)) - 1) << m
```

Where:
- `m` = starting bit position (inclusive, lower)
- `n` = ending bit position (inclusive, upper)
- `n - m + 1` = number of bits in the range

### Step-by-Step Derivation

The formula works in two stages:

**Stage 1** — Create a block of consecutive `1`s starting at bit 0:

The expression `(1 << k) - 1` produces `k` consecutive `1`s. Here is why:

```
  1 << 4       = 00010000    (a single 1 in position 4)
  (1 << 4) - 1 = 00001111    (subtracting 1 flips all bits below position 4)
                   ^^^^
                   four consecutive 1s
```

**Stage 2** — Shift the block into position:

Shifting left by `m` moves the block so it starts at bit `m`:

```
  00001111 << 2 = 00111100   (the block now covers bits 2-5)
```

### Full Example: Mask for Bits 2 through 5

```
  m = 2, n = 5
  Width = n - m + 1 = 5 - 2 + 1 = 4

  Step 1:  1 << 4           = 00010000   (16)
  Step 2:  (1 << 4) - 1     = 00001111   (15)  four 1s at bottom
  Step 3:  00001111 << 2    = 00111100   (60)  shifted into position

  Final mask: 00111100
              ^^^^
              bits 5,4,3,2 are all set
```

```ad-warning
title: Watch the operator precedence
In C, C++, C#, and JavaScript, the shift operators `<<` and `>>` have **lower** precedence than `+` and `-`. You must use parentheses:

Correct:  `((1 << (n - m + 1)) - 1) << m`
Wrong:    `1 << n - m + 1 - 1 << m`   (parsed completely differently)

When in doubt, add parentheses. They cost nothing at runtime and prevent subtle bugs.
```

```ad-tip
title: Alternative range mask formulas
Some programmers use different but equivalent expressions:

- `~(~0 << (n - m + 1)) << m` — builds the block by inverting a shifted all-ones
- `((1 << (n + 1)) - 1) & ~((1 << m) - 1)` — builds separate upper and lower masks and combines them

All three produce the same result. Use whichever reads most clearly to you and your team.
```

---

## Special Masks

### All-Ones Mask

The expression `~0` flips every bit of zero, producing a value where **every bit is `1`**. This is width-agnostic — it works for 8-bit, 16-bit, 32-bit, and 64-bit types.

```
  0    = 00000000 00000000 00000000 00000000   (32-bit)
  ~0   = 11111111 11111111 11111111 11111111   (32-bit)
```

You can also write `0xFFFFFFFF` for an explicit 32-bit all-ones value, but `~0` adapts to the type's width automatically.

```ad-tip
title: When to use ~0 vs explicit hex
Use `~0` when you want "all bits set, whatever the width." Use `0xFF`, `0xFFFF`, or `0xFFFFFFFF` when you need a specific number of bits set. Mixing them up is a common source of bugs when porting code between 32-bit and 64-bit platforms.
```

### All-Zeros Mask

Simply `0`. ANDing with zero clears everything; ORing with zero changes nothing. While trivial, it is worth noting because `0` is the identity element for `|` and the annihilator for `&`.

### Inverted Masks

The [[NOT Operator]] `~` inverts every bit in a mask. This is most commonly used to create a "clear mask" from a "set mask":

```
  mask  = 00001000    (targets bit 3)
  ~mask = 11110111    (targets everything EXCEPT bit 3)

  value & ~mask       clears bit 3, leaves all others intact
```

See [[Check Set and Clear Bits]] for the full set/clear/toggle patterns using these masks.

---

## Hex Notation for Common Masks

Experienced developers often write masks in hexadecimal rather than binary because hex is more compact and each hex digit maps directly to 4 binary digits (one nibble). See [[Hexadecimal and Octal]] for the full conversion table.

### Byte and Nibble Masks

| Hex      | Binary (16-bit)        | Description             |
|----------|------------------------|-------------------------|
| `0x00FF` | `00000000 11111111`    | Low byte (bits 0-7)     |
| `0xFF00` | `11111111 00000000`    | High byte (bits 8-15)   |
| `0x0F`   | `00001111`             | Low nibble (bits 0-3)   |
| `0xF0`   | `11110000`             | High nibble (bits 4-7)  |

### 32-Bit Byte Extraction Masks

| Hex          | Bits Selected | Common Use              |
|--------------|---------------|-------------------------|
| `0x000000FF` | 0-7           | Extract byte 0 (lowest) |
| `0x0000FF00` | 8-15          | Extract byte 1          |
| `0x00FF0000` | 16-23         | Extract byte 2          |
| `0xFF000000` | 24-31         | Extract byte 3 (highest)|

### Color Channel Masks (ARGB)

A practical example: ARGB color values pack four 8-bit channels into a 32-bit integer:

```
  0xAARRGGBB

  Alpha mask: 0xFF000000   (bits 24-31)
  Red mask:   0x00FF0000   (bits 16-23)
  Green mask: 0x0000FF00   (bits 8-15)
  Blue mask:  0x000000FF   (bits 0-7)
```

```ad-tip
title: Reading hex masks at a glance
Each hex digit is exactly 4 bits. `F` = all four bits set, `0` = all four bits clear. So `0x0F` means "bottom 4 bits on, top 4 bits off" — you can read it instantly without converting to binary.
```

---

## Code Examples

### C#

```csharp
using System;

class BitMaskDemo
{
    static void Main()
    {
        // --- Single-bit mask ---
        int singleMask = 1 << 3;  // 00001000 (bit 3)
        Console.WriteLine($"Single mask:  {Convert.ToString(singleMask, 2).PadLeft(8, '0')}");

        // --- Multi-bit mask ---
        int multiMask = (1 << 2) | (1 << 5);  // 00100100 (bits 2 and 5)
        Console.WriteLine($"Multi mask:   {Convert.ToString(multiMask, 2).PadLeft(8, '0')}");

        // --- Range mask for bits 2 through 5 ---
        int m = 2, n = 5;
        int rangeMask = ((1 << (n - m + 1)) - 1) << m;  // 00111100
        Console.WriteLine($"Range mask:   {Convert.ToString(rangeMask, 2).PadLeft(8, '0')}");

        // --- Inverted mask ---
        // Cast to byte-width for display; ~singleMask in 32-bit would show 24 leading 1s
        int inverted = ~singleMask & 0xFF;  // 11110111
        Console.WriteLine($"Inverted:     {Convert.ToString(inverted, 2).PadLeft(8, '0')}");

        // --- All-ones mask (32-bit) ---
        uint allOnes = unchecked((uint)~0);  // 0xFFFFFFFF
        Console.WriteLine($"All ones:     0x{allOnes:X8}");

        // --- Byte extraction from a 32-bit color ---
        uint color = 0xFF8040C0;  // ARGB
        uint alpha = (color >> 24) & 0xFF;
        uint red   = (color >> 16) & 0xFF;
        uint green = (color >> 8)  & 0xFF;
        uint blue  = color & 0xFF;
        Console.WriteLine($"A={alpha} R={red} G={green} B={blue}");
    }
}
```

### C++

```cpp
#include <cstdint>
#include <iostream>
#include <bitset>

int main() {
    // --- Single-bit mask ---
    constexpr uint8_t singleMask = 1 << 3;  // 00001000
    std::cout << "Single mask:  " << std::bitset<8>(singleMask) << "\n";

    // --- Multi-bit mask ---
    constexpr uint8_t multiMask = (1 << 2) | (1 << 5);  // 00100100
    std::cout << "Multi mask:   " << std::bitset<8>(multiMask) << "\n";

    // --- Range mask for bits 2 through 5 ---
    constexpr int m = 2, n = 5;
    constexpr uint8_t rangeMask = ((1 << (n - m + 1)) - 1) << m;  // 00111100
    std::cout << "Range mask:   " << std::bitset<8>(rangeMask) << "\n";

    // --- Inverted mask ---
    constexpr uint8_t inverted = static_cast<uint8_t>(~singleMask);  // 11110111
    std::cout << "Inverted:     " << std::bitset<8>(inverted) << "\n";

    // --- All-ones mask ---
    constexpr uint32_t allOnes = ~uint32_t(0);  // 0xFFFFFFFF
    std::cout << "All ones:     " << std::bitset<32>(allOnes) << "\n";

    // --- Byte extraction ---
    constexpr uint32_t color = 0xFF8040C0;
    constexpr uint8_t alpha = (color >> 24) & 0xFF;
    constexpr uint8_t red   = (color >> 16) & 0xFF;
    constexpr uint8_t green = (color >> 8)  & 0xFF;
    constexpr uint8_t blue  = color & 0xFF;
    std::cout << "A=" << +alpha << " R=" << +red
              << " G=" << +green << " B=" << +blue << "\n";

    return 0;
}
```

```ad-note
title: The unary + trick in C++
Writing `+alpha` instead of `alpha` when printing a `uint8_t` promotes it to `int`, so `std::cout` prints it as a number instead of interpreting it as a character.
```

### JavaScript

```javascript
// --- Single-bit mask ---
const singleMask = 1 << 3;  // 8
console.log("Single mask:  " + singleMask.toString(2).padStart(8, '0'));
// Output: 00001000

// --- Multi-bit mask ---
const multiMask = (1 << 2) | (1 << 5);  // 36
console.log("Multi mask:   " + multiMask.toString(2).padStart(8, '0'));
// Output: 00100100

// --- Range mask for bits 2 through 5 ---
const m = 2, n = 5;
const rangeMask = ((1 << (n - m + 1)) - 1) << m;  // 60
console.log("Range mask:   " + rangeMask.toString(2).padStart(8, '0'));
// Output: 00111100

// --- Inverted mask (truncated to 8 bits for display) ---
const inverted = (~singleMask) & 0xFF;  // 247
console.log("Inverted:     " + inverted.toString(2).padStart(8, '0'));
// Output: 11110111

// --- All-ones mask ---
// JavaScript bitwise ops use 32-bit signed integers
const allOnes = ~0;          // -1 in signed, but all 32 bits are 1
const unsigned = allOnes >>> 0;  // convert to unsigned: 4294967295
console.log("All ones:     0x" + unsigned.toString(16).toUpperCase());
// Output: 0xFFFFFFFF

// --- Helper: display any value as 8-bit binary ---
function toBin8(val) {
    return (val & 0xFF).toString(2).padStart(8, '0');
}
console.log("Bit 3 mask:   " + toBin8(1 << 3));   // 00001000
console.log("Bits 2+5:     " + toBin8(multiMask)); // 00100100
```

```ad-warning
title: JavaScript's 32-bit limitation
All JavaScript bitwise operators coerce their operands to **32-bit signed integers**. This means `1 << 31` produces `-2147483648` (the sign bit), and you cannot create masks wider than 32 bits with `<<`. For wider masks, use `BigInt`:
```javascript
const wideMask = 1n << 40n;  // BigInt literal
```
```

---

## Common Mistakes

```ad-warning
title: Shifting by the type's width or more
Shifting by an amount equal to or greater than the bit-width of the type is **undefined behavior** in C/C++ and produces unexpected results in other languages. For a 32-bit `int`, `1 << 32` is undefined in C++, wraps to `1` in C# (shift amount is masked mod 32), and yields `0` in some other contexts. Always ensure your shift amount is in the range `[0, width - 1]`.
```

```ad-warning
title: Signed integer shifts
Using `1 << 31` on a 32-bit `int` sets the sign bit, producing a negative number. If you need to set bit 31, use an unsigned type:
- C++: `1u << 31` or `uint32_t(1) << 31`
- C#: `1u << 31` (unsigned literal)
- JavaScript: use `>>> 0` to interpret as unsigned, or use `BigInt`
```

```ad-warning
title: Forgetting parentheses in range mask formula
The expression `1 << n - m + 1 - 1 << m` does not do what you think. Due to operator precedence, the subtraction binds tighter than the shift. Always parenthesize: `((1 << (n - m + 1)) - 1) << m`.
```

---

## See Also

- [[Left Shift]] — the shift operator used to build masks
- [[AND Operator]] — used with masks to check and clear bits
- [[OR Operator]] — used with masks to set bits and combine masks
- [[XOR Operator]] — used with masks to toggle bits
- [[NOT Operator]] — used to invert masks
- [[Hexadecimal and Octal]] — compact notation for writing masks
- [[Check Set and Clear Bits]] — applying masks to read and modify individual bits
- [[Flag Enums and Bit Flags]] — using masks in enum-based flag systems
