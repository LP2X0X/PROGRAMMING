---
title: Right Shift
date: 2026-08-19
tags:
  - bit-manipulation
  - operator
  - shift
aliases:
  - right shift operator
  - arithmetic right shift
  - bitwise right shift
---

## Right Shift Operator (`>>`)

The **right shift operator** (`>>`) shifts all [[Bits Bytes and Words|bits]] in a value to the right by a specified number of positions. Each shift to the right effectively moves every bit one place toward the least significant bit (LSB). Bits that "fall off" the right end are discarded. What fills in on the left side depends on whether the shift is **arithmetic** or **logical** — which in turn depends on the signedness of the operand and the language you are working in.

Right shift is the mirror counterpart of [[Left Shift]], which moves bits in the opposite direction. JavaScript also has a distinct [[Unsigned Right Shift]] operator (`>>>`) that always performs a logical shift regardless of sign.

```
Syntax:   result = value >> n
```

Where `n` is the number of positions to shift.

---

## How It Works — Bit-Level View

When you right-shift a value by `n`, every bit moves `n` positions to the right. The `n` least significant bits are lost, and `n` new bits appear on the left. The value of those new bits is the key distinction between the two flavors of right shift.

### ASCII Diagram — Arithmetic Right Shift (Signed)

```
Original (int8, value = -24, binary in two's complement):

  MSB                         LSB
  [1][1][1][0][1][0][0][0]       = -24

Right shift by 2  (>> 2):

  Sign bit is replicated -->
  [1][1] are inserted on the left
                     [0][0] fall off the right

  [1][1][1][1][1][0][1][0]       = -6
   ^  ^
   |  |
   Replicated sign bits
```

### ASCII Diagram — Logical Right Shift (Unsigned)

```
Original (uint8, value = 232, binary):

  MSB                         LSB
  [1][1][1][0][1][0][0][0]       = 232

Right shift by 2  (>> 2):

  Zeros are inserted on the left -->
  [0][0] are inserted
                     [0][0] fall off the right

  [0][0][1][1][1][0][1][0]       = 58
   ^  ^
   |  |
   Zero-filled
```

---

## Arithmetic vs. Logical Right Shift

There are two fundamentally different behaviors a right shift can exhibit, and understanding the distinction is critical when working with [[Signed and Unsigned Integers]].

### Arithmetic Right Shift

- The **sign bit** (MSB) is **replicated** into the vacated positions on the left.
- Negative numbers stay negative after shifting.
- This is the default for **signed** integer types in most languages.
- Preserves the [[Twos Complement]] representation of the sign.

### Logical Right Shift

- **Zeros** are always inserted into the vacated positions on the left.
- The result is always non-negative (for the same bit width).
- This is the default for **unsigned** integer types.
- JavaScript provides a dedicated logical right shift operator: [[Unsigned Right Shift|`>>>`]].

```ad-note
title: Why Two Flavors?
collapse: open
Arithmetic right shift exists so that shifting a [[Twos Complement]] signed integer right by 1 correctly approximates division by 2 for both positive and negative values. If zeros were always inserted (logical shift), a negative number would become a large positive number — completely wrong for signed arithmetic.
```

---

## Equivalence to Division by 2^n

Right-shifting by `n` is equivalent to integer division by `2^n`:

```
value >> n   ≈   value / (2^n)
```

For **positive** values, this equivalence is exact (the fractional part is truncated, same as integer division).

For **negative** values, there is a subtle but important difference:

```ad-warning
title: Rounding Direction for Negative Numbers
collapse: open
Arithmetic right shift rounds toward **negative infinity** (floors the result), while regular integer division in most languages rounds toward **zero** (truncates).

Example with -7:
  -7 >> 1  =  -4     (rounds toward negative infinity)
  -7 / 2   =  -3     (rounds toward zero in C#, C++, JS)

This difference matters whenever you use right shift as an optimization for division on values that may be negative. They are NOT interchangeable for negative operands.
```

| Expression | Result | Rounding direction |
|---|---|---|
| `7 >> 1` | `3` | Toward zero (same either way) |
| `7 / 2` | `3` | Toward zero |
| `-7 >> 1` | `-4` | Toward negative infinity |
| `-7 / 2` | `-3` | Toward zero |

---

## Language-Specific Behavior

The behavior of `>>` varies meaningfully across C++, C#, and JavaScript. Getting this wrong is a common source of bugs, especially when porting code between languages.

### C++

- For **unsigned** types (`unsigned int`, `uint32_t`, etc.): logical shift (zeros inserted). This is guaranteed by the standard.
- For **signed** types (`int`, `int32_t`, etc.) with **non-negative** values: logical shift (zeros inserted). Guaranteed.
- For **signed** types with **negative** values: the behavior is **implementation-defined**.

```ad-warning
title: C++ and Negative Right Shift
collapse: open
The C++ standard (up through C++20) says that right-shifting a negative signed integer is **implementation-defined**. In practice, virtually every mainstream compiler (GCC, Clang, MSVC) performs an arithmetic shift (sign-extending), but you cannot portably rely on this. If you need guaranteed arithmetic right shift on negative values in C++, you should document the assumption or use compiler-specific intrinsics.

Starting with C++20, signed integers are required to use [[Twos Complement]] representation, but the right-shift behavior for negative values remains implementation-defined until C++23, which finally guarantees arithmetic right shift for signed types.
```

### C\#

C# has well-defined, predictable behavior:

- **Signed types** (`int`, `long`, `short`, `sbyte`): arithmetic right shift. The sign bit is always preserved.
- **Unsigned types** (`uint`, `ulong`, `ushort`, `byte`): logical right shift. Zeros are always inserted.

There is no ambiguity or implementation-defined behavior. The language specification guarantees these semantics.

### JavaScript

- `>>` is **always arithmetic** (sign-preserving). The operand is converted to a 32-bit signed integer before shifting.
- [[Unsigned Right Shift|`>>>`]] is **always logical** (zero-filling). The operand is converted to a 32-bit unsigned integer before shifting.
- JavaScript does not have separate signed/unsigned integer types, so it provides two distinct operators instead.

| Language | Signed negative `>>` | Unsigned `>>` | Separate logical shift operator |
|---|---|---|---|
| C++ | Implementation-defined (usually arithmetic) | Logical (guaranteed) | No |
| C# | Arithmetic (guaranteed) | Logical (guaranteed) | `>>>` added in C# 11 |
| JavaScript | Arithmetic (guaranteed) | N/A (no unsigned int type) | `>>>` |

---

## Shift Count Handling

What happens when the shift count is very large, zero, or negative varies by language, and getting it wrong leads to subtle bugs.

### C\#

The shift count is **masked** to the valid range for the type:

- For `int` and `uint` (32-bit): the count is masked with `& 0x1F` (only the low 5 bits matter), giving an effective range of 0-31.
- For `long` and `ulong` (64-bit): the count is masked with `& 0x3F` (only the low 6 bits matter), giving an effective range of 0-63.

```ad-tip
title: Masking Implication
collapse: open
This means `x >> 32` is equivalent to `x >> 0` for a 32-bit int in C# — the value is unchanged. The shift count 32 is masked to 0. This can be surprising if you expect it to zero out the value.
```

### C++

- Shifting by a count of **0** is a no-op (returns the original value).
- Shifting by a count **greater than or equal to the bit width** of the type is **undefined behavior**. The compiler can do anything — it might return zero, the original value, or cause a crash.
- Shifting by a **negative** count is also undefined behavior.

```ad-warning
title: Undefined Behavior in C++
collapse: open
Never shift by an amount >= the bit width of the type in C++. For a 32-bit int, `x >> 32` is undefined behavior, NOT guaranteed to produce 0. Always validate or clamp shift counts when they come from user input or computation.
```

### JavaScript

- The shift count is converted to a **uint32** and then masked with `& 0x1F`, giving an effective range of 0-31.
- The operand is always converted to a **32-bit signed integer** before the arithmetic shift.
- Negative shift counts wrap around: `-1 & 0x1F = 31`, so `x >> -1` is equivalent to `x >> 31`.

---

## Code Examples

### C\#

```csharp
using System;

class RightShiftDemo
{
    static void Main()
    {
        // --- Positive signed value ---
        int positive = 40;               // 00000000_00000000_00000000_00101000
        int result1 = positive >> 2;      // 00000000_00000000_00000000_00001010
        Console.WriteLine($"40 >> 2 = {result1}");   // 10  (40 / 4)

        // --- Negative signed value (arithmetic shift) ---
        int negative = -24;              // 11111111_11111111_11111111_11101000
        int result2 = negative >> 2;     // 11111111_11111111_11111111_11111010
        Console.WriteLine($"-24 >> 2 = {result2}");  // -6  (sign preserved)

        // --- Unsigned value (logical shift) ---
        uint unsigned_val = 0xFF000000;  // 11111111_00000000_00000000_00000000
        uint result3 = unsigned_val >> 4;// 00001111_11110000_00000000_00000000
        Console.WriteLine($"0xFF000000 >> 4 = 0x{result3:X8}");  // 0x0FF00000

        // --- Shift count masking ---
        int val = 100;
        int result4 = val >> 32;  // Masked: 32 & 0x1F = 0, so no shift
        Console.WriteLine($"100 >> 32 = {result4}");  // 100 (no shift occurred)

        // --- Division equivalence caveat ---
        int neg7 = -7;
        Console.WriteLine($"-7 >> 1 = {neg7 >> 1}");  // -4 (floor toward -inf)
        Console.WriteLine($"-7 / 2  = {neg7 / 2}");   // -3 (truncate toward 0)
    }
}
```

### C++

```cpp
#include <iostream>
#include <cstdint>

int main() {
    // --- Positive signed value ---
    int positive = 40;               // 00000000...00101000
    int result1 = positive >> 2;     // 00000000...00001010
    std::cout << "40 >> 2 = " << result1 << "\n";   // 10

    // --- Negative signed value ---
    // Implementation-defined, but arithmetic shift on most compilers
    int negative = -24;
    int result2 = negative >> 2;
    std::cout << "-24 >> 2 = " << result2 << "\n";  // -6 (on most compilers)

    // --- Unsigned value (guaranteed logical shift) ---
    uint32_t unsigned_val = 0xFF000000u;
    uint32_t result3 = unsigned_val >> 4;
    std::cout << "0xFF000000 >> 4 = " << std::hex << result3 << "\n";  // 0ff00000

    // --- DANGER: shifting by >= bit width is UNDEFINED BEHAVIOR ---
    // int bad = 100 >> 32;   // DO NOT DO THIS — undefined behavior!

    // --- Safe approach: validate shift count ---
    int shift_count = 32;
    int val = 100;
    int safe_result = (shift_count >= 32) ? 0 : (val >> shift_count);
    std::cout << std::dec << "Safe shift: " << safe_result << "\n";  // 0

    return 0;
}
```

### JavaScript

```javascript
// --- Positive value ---
let positive = 40;
console.log(`40 >> 2 = ${positive >> 2}`);       // 10

// --- Negative value (arithmetic shift, sign preserved) ---
let negative = -24;
console.log(`-24 >> 2 = ${negative >> 2}`);      // -6

// --- Comparison: >> vs >>> ---
console.log(`-1 >> 1  = ${-1 >> 1}`);            // -1  (arithmetic, all 1s stay)
console.log(`-1 >>> 1 = ${-1 >>> 1}`);           // 2147483647  (logical, 0 inserted)

// --- Shift count masking ---
console.log(`100 >> 32 = ${100 >> 32}`);         // 100  (32 & 0x1F = 0)
console.log(`100 >> 33 = ${100 >> 33}`);         // 50   (33 & 0x1F = 1)

// --- Negative shift count wraps ---
console.log(`100 >> -1 = ${100 >> -1}`);         // 0    (-1 & 0x1F = 31)

// --- Division equivalence caveat ---
console.log(`-7 >> 1      = ${-7 >> 1}`);        // -4 (floor)
console.log(`Math.trunc(-7/2) = ${Math.trunc(-7/2)}`);  // -3 (truncate)
```

---

## Common Patterns and Use Cases

Right shift appears frequently in low-level and performance-sensitive code:

- **Fast division by powers of 2**: `x >> 3` instead of `x / 8` (but beware the negative-number caveat).
- **Extracting bit fields**: shift a value right to move the target bits to the LSB positions, then mask with [[AND Operator|`&`]] to isolate them.
  ```csharp
  // Extract bits 4-7 from a byte
  int bits4to7 = (value >> 4) & 0x0F;
  ```
- **Sign extension**: arithmetic right shift by the full width minus 1 produces a mask of all 1s (for negative) or all 0s (for non-negative).
  ```csharp
  int sign_mask = value >> 31;  // 0xFFFFFFFF if negative, 0x00000000 if non-negative
  ```
- **Color channel extraction**: in graphics and image processing, packed pixel values are shifted right to isolate individual channels.
  ```csharp
  // Extract green channel from 0xAARRGGBB
  int green = (pixel >> 8) & 0xFF;
  ```
- **Average without overflow**: `(a + b) >> 1` is sometimes used, but note this can overflow if `a + b` exceeds the type range. A safe alternative is `(a >> 1) + (b >> 1) + ((a & b) & 1)`.

---

## Common Pitfalls

```ad-warning
title: Pitfalls to Watch For
collapse: open
1. **Assuming arithmetic shift in C++**: It works on your compiler today, but it is not guaranteed by the standard (pre-C++23). Do not rely on it in portable code.

2. **Using right shift for division on negative numbers**: The rounding direction differs from integer division. `-7 >> 1` gives `-4`, not `-3`.

3. **Shifting by the full bit width**: In C++ this is undefined behavior. In C# and JavaScript the count is masked, so `x >> 32` for a 32-bit int gives `x` unchanged — not 0.

4. **Forgetting sign extension**: Right-shifting a negative signed value fills with 1s (arithmetic shift), which may produce unexpected results if you later interpret the value as unsigned or use it in a bitmask.

5. **Confusing `>>` with `>>>` in JavaScript**: `>>` preserves the sign, `>>>` does not. Using the wrong one on negative values produces wildly different results.
```

---

## Relationship to Other Operators

| Operator | Direction | Fill behavior | See also |
|---|---|---|---|
| `<<` | Left | Zeros always fill on the right | [[Left Shift]] |
| `>>` | Right | Sign bit (signed) or zeros (unsigned) | This note |
| `>>>` | Right | Zeros always fill on the left | [[Unsigned Right Shift]] |

Right shift is often combined with the [[AND Operator]] to extract specific bit fields from a value. The typical pattern is: shift right to align the desired bits with position 0, then AND with a mask to isolate them.

Understanding right shift deeply requires familiarity with [[Twos Complement]] representation (for why sign extension works) and the [[Binary Number System]] (for why shifting corresponds to division by powers of 2).

---

## Related Topics

- [[Left Shift]]
- [[Unsigned Right Shift]]
- [[Signed and Unsigned Integers]]
- [[Twos Complement]]
- [[Binary Number System]]
- [[Bits Bytes and Words]]
- [[AND Operator]]
