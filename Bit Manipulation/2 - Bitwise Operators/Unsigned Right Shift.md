---
title: "Unsigned Right Shift"
date: 2026-08-19
tags:
  - bit-manipulation
  - operator
  - shift
  - unsigned
aliases:
  - "Zero-Fill Right Shift"
  - "Logical Right Shift"
  - ">>>"
status: complete
---

## Unsigned Right Shift (`>>>`)

```ad-note
title: Overview
The **unsigned right shift** (also called **zero-fill right shift** or **logical right shift**) shifts all bits to the right by a specified number of positions and **always fills the vacated high-order bits with zeros**, regardless of the original sign bit. This contrasts with the [[Right Shift|arithmetic right shift]] (`>>`), which preserves the sign bit. The operator symbol is `>>>` in languages that support it natively.
```

---

## Table of Contents

- [[#How It Works]]
- [[#Arithmetic vs Unsigned Right Shift]]
- [[#ASCII Diagram — Negative Number Shift Comparison]]
- [[#Language Support]]
- [[#When to Use Unsigned vs Arithmetic Right Shift]]
- [[#Code Examples]]
- [[#Edge Cases and Gotchas]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## How It Works

The unsigned right shift operation takes a value and moves every bit to the right by a given count. The key behavior that distinguishes it from the arithmetic [[Right Shift]] is what happens to the **vacated positions** on the left side:

- **Unsigned right shift (`>>>`)**: Always inserts **zeros** into the vacated high-order bits
- **Arithmetic right shift (`>>`)**: Inserts copies of the **sign bit** (the most significant bit) into the vacated positions

For **positive numbers**, both operators produce identical results because the sign bit is already `0`. The difference only manifests with **negative numbers**, where the sign bit is `1` in [[Twos Complement]] representation.

Given a value and a shift count `n`:

1. All bits move `n` positions to the right
2. The `n` lowest bits are discarded (shifted out)
3. The `n` highest bits are filled with `0` (always, unconditionally)

```ad-tip
title: Mental Model
Think of unsigned right shift as "I don't care about the sign — I'm treating this as a raw bit pattern, not a number." The moment you zero-fill, you lose the sign information permanently.
```

---

## Arithmetic vs Unsigned Right Shift

Understanding the contrast between these two shifts is essential. They differ in exactly one way: what fills the vacated high-order bits.

| Property                        | Arithmetic (`>>`)                              | Unsigned (`>>>`)                   |
| ------------------------------- | ---------------------------------------------- | ---------------------------------- |
| Vacated bits filled with        | Copy of sign bit (sign extension)              | Always zero                        |
| Positive number result          | Same as unsigned shift                         | Same as arithmetic shift           |
| Negative number result          | Remains negative (sign preserved)              | Becomes positive (sign destroyed)  |
| Semantic meaning                | Division by 2^n (rounding toward negative inf) | Logical bit pattern manipulation   |
| Preserves [[Twos Complement]]   | Yes                                            | No                                 |

```ad-warning
title: Critical Distinction
When shifting a **negative** 32-bit integer right by 1 using arithmetic shift, the result is still negative. When using unsigned right shift, the result is a large **positive** number because the sign bit is replaced with zero. These are fundamentally different operations on negative values.
```

---

## ASCII Diagram — Negative Number Shift Comparison

Consider the 8-bit [[Twos Complement]] representation of **-20** (`11101100`) shifted right by 2:

```
Original value: -20
Binary (8-bit two's complement): 1 1 1 0 1 1 0 0
                                  ^
                                  sign bit (1 = negative)

=== Arithmetic Right Shift (>> 2) ===

  1 1 1 0 1 1 0 0       original
  ├─┘ │ │ │ │ │ │
  │   ├─┘ │ │ │ │
  │   │   ├─┘ │ │
  │   │   │   ├─┘
  │   │   │   │   ──► lowest 2 bits discarded
  ▼   ▼   ▼   ▼
  1 1 1 1 1 0 1 1       result
  ^^^
  sign bit COPIED into vacated positions
  
  Result: 11111011 = -5  (sign preserved, like dividing by 4)


=== Unsigned Right Shift (>>> 2) ===

  1 1 1 0 1 1 0 0       original
  ├─┘ │ │ │ │ │ │
  │   ├─┘ │ │ │ │
  │   │   ├─┘ │ │
  │   │   │   ├─┘
  │   │   │   │   ──► lowest 2 bits discarded
  ▼   ▼   ▼   ▼
  0 0 1 1 1 0 1 1       result
  ^^^
  ZEROS always fill vacated positions
  
  Result: 00111011 = 59  (sign destroyed, treated as bit pattern)
```

```ad-note
title: Scale This to 32-bit
In practice, integers are 32 or 64 [[Bits Bytes and Words|bits]] wide. With a 32-bit negative number, an unsigned right shift by 1 turns a negative value into a very large positive number (above 2 billion for `int32`). The diagram above uses 8 bits for clarity, but the principle is identical at any width.
```

---

## Language Support

Not all languages provide a dedicated `>>>` operator. Here is the landscape across three major languages:

### JavaScript

JavaScript has had `>>>` since the beginning of the language. In fact, JavaScript's handling of bitwise operations is unique:

- All bitwise operators internally convert operands to **32-bit signed integers**
- The `>>>` operator is the **only** bitwise operator that returns an **unsigned** 32-bit integer result
- `>>> 0` is a common idiom to convert a value to an unsigned 32-bit integer without actually shifting

```ad-tip
title: JavaScript Quirk
`x >>> 0` does not shift any bits — it converts `x` to an unsigned 32-bit integer. This is frequently used to ensure array indices or lengths are non-negative unsigned values. You will see this in library code and polyfills.
```

### C# (C# 11 / .NET 7+)

C# added the `>>>` operator in **C# 11** (released with .NET 7 in November 2022):

- Before C# 11, achieving unsigned right shift required **casting to an unsigned type** (`uint`, `ulong`), shifting with `>>`, then casting back
- The `>>>` operator works on `int`, `long`, `nint`, and their unsigned counterparts
- C# also added the `>>>=` compound assignment operator

```ad-warning
title: Pre-C# 11 Workaround Required
If you are targeting .NET 6 or earlier (or using an older C# language version), the `>>>` operator is **not available**. You must use the cast-to-unsigned pattern shown in the code examples below.
```

### C++

C++ does **not** have a `>>>` operator and there are no proposals to add one:

- The `>>` operator on **unsigned types** already performs a logical (zero-fill) right shift
- The `>>` operator on **signed types** is implementation-defined in older standards (C++17 and before) and performs arithmetic shift in C++20+
- To achieve unsigned right shift in C++, **cast the value to an unsigned type first**, then use `>>`

```ad-note
title: C++ Standard Clarification
Prior to C++20, the behavior of `>>` on signed negative integers was **implementation-defined** — the compiler could choose arithmetic or logical shift. In practice, virtually every compiler used arithmetic shift, but it was not guaranteed. C++20 formally mandates [[Twos Complement]] representation and arithmetic right shift for signed types.
```

| Language   | `>>>` Available | Since             | Workaround                              |
| ---------- | --------------- | ----------------- | --------------------------------------- |
| JavaScript | Yes             | Always (ES1+)     | N/A                                     |
| C#         | Yes             | C# 11 / .NET 7    | Cast to `uint`/`ulong`, then use `>>`   |
| C++        | No              | N/A               | `static_cast<unsigned>()`, then use `>>` |

---

## When to Use Unsigned vs Arithmetic Right Shift

Choosing the correct shift operator depends on **what the bits represent** to you at the moment of the operation.

### Use Unsigned Right Shift (`>>>`) When:

- You are treating the value as a **raw bit pattern**, not as a signed number
- Computing **hash codes** — you want bits to distribute evenly, not sign-extend
- Implementing **checksums** or **CRC calculations**
- Performing **[[Color and Graphics|color manipulation]]** — extracting ARGB channels from a packed 32-bit color value
- Implementing **bit-level algorithms** (e.g., finding highest set bit, population count helpers)
- Working with **protocol fields** that are defined as unsigned
- Dividing an **unsigned quantity** by a power of 2

### Use Arithmetic Right Shift (`>>`) When:

- The value represents a **signed number** and you want to preserve the sign
- You want the effect of **integer division by 2^n** (rounding toward negative infinity)
- You are implementing **sign extension** deliberately
- Working with **audio samples** or **sensor data** that use signed representation

```ad-tip
title: Rule of Thumb
Ask yourself: "Am I working with a **number** or a **bit pattern**?" If the answer is a number and the sign matters, use arithmetic shift (`>>`). If it is a bit pattern, use unsigned shift (`>>>`). When in doubt with unsigned data, use unsigned shift — sign extension on a value you intended as unsigned is a subtle bug.
```

---

## Code Examples

### JavaScript

```javascript
// === Basic Unsigned Right Shift ===
let positive = 32;
console.log(positive >> 2);   // 8  (arithmetic, same result for positive)
console.log(positive >>> 2);  // 8  (unsigned, same result for positive)

// === Negative Number — Here the Difference Shows ===
let negative = -20;
console.log(negative >> 2);   // -5   (sign preserved, arithmetic shift)
console.log(negative >>> 2);  // 1073741819  (sign destroyed, zero-filled)

// Explanation:
// -20 in 32-bit two's complement:
//   11111111 11111111 11111111 11101100
// Arithmetic >> 2:
//   11111111 11111111 11111111 11111011  = -5
// Unsigned >>> 2:
//   00111111 11111111 11111111 11111011  = 1073741819

// === Common Idiom: Convert to Unsigned 32-bit ===
let val = -1;
console.log(val >>> 0);  // 4294967295  (0xFFFFFFFF, all 32 bits set)

// === Extracting Color Channels from ARGB ===
let color = 0xFF8040C0;  // ARGB packed color
let alpha = (color >>> 24) & 0xFF;  // 255
let red   = (color >>> 16) & 0xFF;  // 128
let green = (color >>> 8)  & 0xFF;  // 64
let blue  =  color         & 0xFF;  // 192
```

### C#

```csharp
// === C# 11+ (.NET 7+) — Native >>> Operator ===
int positive = 32;
Console.WriteLine(positive >> 2);   // 8
Console.WriteLine(positive >>> 2);  // 8  (same for positive values)

int negative = -20;
Console.WriteLine(negative >> 2);   // -5   (arithmetic, sign preserved)
Console.WriteLine(negative >>> 2);  // 1073741819  (unsigned, zero-filled)

// === Pre-C# 11 Workaround — Cast to uint ===
int value = -20;

// Step 1: Reinterpret the bit pattern as unsigned
// Step 2: Shift (>> on uint is always logical/zero-fill)
// Step 3: Cast back to int if needed
int unsignedShifted = (int)((uint)value >> 2);
Console.WriteLine(unsignedShifted);  // 1073741819

// For 64-bit values, use ulong:
long longVal = -20L;
long longResult = (long)((ulong)longVal >> 2);

// === Practical Example: Unsigned Shift for Hash Mixing ===
int hash = someObject.GetHashCode();
// Spread bits using unsigned shift to avoid sign-extension artifacts
hash ^= hash >>> 16;  // C# 11+
// Pre-C# 11 equivalent:
hash ^= (int)((uint)hash >> 16);
```

### C++

```cpp
#include <cstdint>
#include <iostream>

int main() {
    // === C++ Has No >>> Operator ===
    // >> on signed types = arithmetic shift (C++20 guarantees this)
    // >> on unsigned types = logical shift (always zero-fill)

    int32_t negative = -20;

    // Arithmetic right shift (sign-preserving)
    int32_t arithResult = negative >> 2;
    std::cout << arithResult << std::endl;  // -5

    // Unsigned right shift — cast to unsigned first
    int32_t unsignedResult = static_cast<int32_t>(
        static_cast<uint32_t>(negative) >> 2
    );
    std::cout << unsignedResult << std::endl;  // 1073741819

    // === Explanation of the Cast Pattern ===
    // 1. static_cast<uint32_t>(negative)
    //    Reinterprets the bit pattern as unsigned.
    //    -20 (signed) -> 4294967276 (unsigned), same bits: 0xFFFFFFEC
    //
    // 2. >> 2 on uint32_t
    //    Logical shift: zeros fill from the left
    //    0xFFFFFFEC >> 2 = 0x3FFFFFFB = 1073741819
    //
    // 3. static_cast<int32_t>(...)
    //    Cast back to signed if the rest of your code expects int

    // === Helper Function for Clarity ===
    // You can wrap this in a utility function:
    auto unsigned_right_shift = [](int32_t val, int count) -> int32_t {
        return static_cast<int32_t>(static_cast<uint32_t>(val) >> count);
    };

    std::cout << unsigned_right_shift(-1, 1) << std::endl;  // 2147483647

    // === Extracting Bytes from a Packed Value ===
    uint32_t color = 0xFF8040C0;
    uint8_t alpha = (color >> 24) & 0xFF;  // 255
    uint8_t red   = (color >> 16) & 0xFF;  // 128
    uint8_t green = (color >> 8)  & 0xFF;  // 64
    uint8_t blue  =  color        & 0xFF;  // 192
    // When the variable is already unsigned, >> is already a logical shift.
    // No special handling needed.

    return 0;
}
```

```ad-warning
title: C++ Pitfall — Shift Width
In all three languages, shifting by a count greater than or equal to the bit width of the type is **undefined behavior** in C++ and produces surprising results in JavaScript and C#. For a 32-bit integer, valid shift counts are 0 through 31. JavaScript masks the shift count to 5 bits (`count & 0x1F`), so `x >>> 32` is equivalent to `x >>> 0` (no shift at all). C# similarly masks to 5 bits for `int` and 6 bits for `long`.
```

---

## Edge Cases and Gotchas

### Shifting Zero

`0 >>> n` is always `0` for any valid shift count. No surprises here.

### Shifting by Zero

`x >>> 0` does not change the bit pattern, but in JavaScript it converts the value to an unsigned 32-bit integer. In C# and C++, it is a no-op.

### The `-1` Test Case

`-1` in [[Twos Complement]] is all `1` bits. This makes it a perfect test value:

| Expression (32-bit) | Result              | Explanation                                    |
| -------------------- | ------------------- | ---------------------------------------------- |
| `-1 >> 1`            | `-1`                | All 1s, sign-extended with 1 — stays all 1s    |
| `-1 >>> 1`           | `2147483647`        | All 1s, zero-filled — becomes `0x7FFFFFFF`     |
| `-1 >>> 16`          | `65535`             | Upper 16 bits zeroed — becomes `0x0000FFFF`    |
| `-1 >>> 31`          | `1`                 | Only the old sign bit remains — becomes `0x01` |

### JavaScript Type Coercion

JavaScript converts all bitwise operands to 32-bit integers. If you have a number larger than 32 bits (which JavaScript numbers can represent as 64-bit floats), the upper bits are silently truncated before the shift occurs.

### Mixing Signed and Unsigned in C++

```ad-warning
title: Implicit Conversion Trap
In C++, mixing signed and unsigned operands in expressions triggers **implicit unsigned conversion**, which can cause unexpected results. If you subtract and the result goes negative, it wraps around to a huge positive number. When using the cast-to-unsigned pattern for shifting, cast back to signed immediately after the shift to avoid propagating unsigned arithmetic through your expressions.
```

### Relationship to [[AND Operator|Masking]]

Unsigned right shift followed by an [[AND Operator|AND mask]] is the standard pattern for extracting bit fields from packed values. For example, extracting bits 8-15 from a 32-bit value:

```javascript
let bits8to15 = (value >>> 8) & 0xFF;
```

The unsigned shift moves the desired bits into the lowest positions, and the AND mask isolates exactly those bits. This is the same pattern used in [[Color and Graphics|color channel extraction]] and protocol parsing.

### Relationship to [[Left Shift]]

[[Left Shift]] (`<<`) always fills vacated low-order bits with zeros regardless of sign. There is no "unsigned left shift" because left shift already behaves uniformly. The signed/unsigned distinction only matters for right shift because only right shift must decide what to do with the sign bit's position.

---

## Comprehensive Summary

```ad-tip
title: Complete Summary
**Unsigned right shift** (`>>>`) moves bits to the right and **always fills vacated high-order bits with zeros**, regardless of whether the original value is positive or negative. This contrasts with arithmetic right shift (`>>`), which copies the sign bit into vacated positions, preserving the sign of the value.

The practical impact is that unsigned right shift on a negative number produces a large positive result (the sign bit becomes `0`), while arithmetic right shift preserves negativity. For positive numbers, both operators produce identical results.

**Language support varies**: JavaScript has always had `>>>`, C# added it in C# 11 (.NET 7), and C++ has no dedicated operator — you must cast to an unsigned type and then use `>>`, which performs a logical shift on unsigned operands.

**Use unsigned right shift** when treating a value as a raw bit pattern rather than a signed number — hash code computation, checksum algorithms, color channel extraction, and protocol field parsing are the primary use cases. **Use arithmetic right shift** when you want sign-preserving division by a power of two.

Key gotchas include: shift counts must be less than the type's bit width, JavaScript's `>>> 0` idiom for unsigned conversion, and C++'s implicit signed-to-unsigned conversion traps when mixing types in expressions.
```

---

## Related Topics

- [[Right Shift]] — the arithmetic (sign-preserving) counterpart
- [[Left Shift]] — always zero-fills, no signed/unsigned distinction
- [[Signed and Unsigned Integers]] — the type system foundation for understanding shift behavior
- [[Twos Complement]] — the representation that makes sign extension meaningful
- [[Binary Number System]] — prerequisite for understanding bit-level operations
- [[AND Operator]] — commonly paired with shifts for bit field extraction
- [[Color and Graphics]] — a major practical application of unsigned shifts
- [[Bits Bytes and Words]] — bit widths that determine shift range and behavior
