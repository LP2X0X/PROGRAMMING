---
title: Left Shift
tags:
  - bit-manipulation
  - operator
  - shift
---

## Left Shift Operator (`<<`)

The **left shift operator** (`<<`) shifts all [[Bits Bytes and Words|bits]] in a value to the left by a specified number of positions. Bits vacated on the right side are filled with zeros, while bits that overflow beyond the high end of the type are discarded permanently.

Left shifting is one of the most commonly used bitwise operations, forming the backbone of techniques like [[Creating Bit Masks|mask creation]], fast arithmetic, and bit field packing.

```
Syntax:   result = value << n
```

Where `n` is the number of positions to shift left.

---

## How Left Shift Works

When you left shift a value by `n` positions, every bit moves `n` places toward the most significant bit (MSB). The `n` lowest bit positions are filled with `0`, and the `n` highest bits are lost — they simply fall off the end.

### ASCII Diagram — Shifting Left by 2

```
Original value (13 = 0b00001101), shift left by 2:

  Discarded                                   Zero-filled
  (fall off)                                  (vacated)
     |                                            |
     v                                            v
  +--+--+--+--+--+--+--+--+                      
  | 0| 0| 0| 0| 1| 1| 0| 1|    Original: 13
  +--+--+--+--+--+--+--+--+
        \  \  \  \  \  \--------->  All bits slide LEFT by 2
         \  \  \  \  \---------->
  +--+--+--+--+--+--+--+--+
  | 0| 0| 1| 1| 0| 1| 0| 0|    Result: 52
  +--+--+--+--+--+--+--+--+
                        ^  ^
                        |  |
                     Filled with 0s
```

Step by step:
1. Every bit shifts 2 positions to the left
2. The two highest bits (`0`, `0`) are pushed out and discarded
3. The two lowest positions are filled with `0`
4. Result: `00110100` = 52

---

## Multiplication by Powers of 2

Left shifting by `n` is mathematically equivalent to multiplying by `2^n` — as long as no significant bits overflow out of the type.

This works because the [[Binary Number System|binary number system]] is positional and base-2. Moving a bit one position to the left doubles its positional value, just as moving a digit left in decimal multiplies it by 10.

| Expression | Equivalent Multiplication | Result |
|---|---|---|
| `5 << 1` | 5 x 2^1 = 5 x 2 | 10 |
| `5 << 2` | 5 x 2^2 = 5 x 4 | 20 |
| `5 << 3` | 5 x 2^3 = 5 x 8 | 40 |
| `3 << 4` | 3 x 2^4 = 3 x 16 | 48 |
| `1 << 10` | 1 x 2^10 = 1 x 1024 | 1024 |

```ad-tip
title: Why This is Faster Than Multiplication
On most processors, a bit shift is a single CPU instruction that completes in one clock cycle. Multiplication, while also fast on modern hardware, may take multiple cycles depending on architecture. Compilers routinely optimize multiplications by powers of 2 into left shifts automatically, so you rarely need to do this manually for performance — but understanding *why* it works is essential for reading low-level code and working with bit fields.
```

### Proof by Example

```
  Decimal 7  =  0b00000111
  
  7 << 1:       0b00001110  =  14  (7 x 2)
  7 << 2:       0b00011100  =  28  (7 x 4)
  7 << 3:       0b00111000  =  56  (7 x 8)
```

Each shift left doubles the value, just as appending a zero in decimal multiplies by 10 (e.g., 7 becomes 70).

---

## Bit Loss (Overflow)

Bits that are shifted past the most significant position of the data type are permanently discarded. This means left shifting can produce unexpected results when high bits contain meaningful data.

```
  8-bit unsigned example:
  
  200  =  0b11001000
  
  200 << 1:
  
    1 1 0 0 1 0 0 0      << 1
   [1] 1 0 0 1 0 0 0 0    the leading '1' falls off
        result: 0b10010000 = 144   (NOT 400)
```

The mathematically correct answer (200 x 2 = 400) exceeds the 8-bit unsigned max of 255, so the high bit is lost. This is ==silent data loss== — no error or exception is raised.

```ad-warning
title: Overflow is Silent
Left shifting never throws an exception or sets an error flag in C++, C#, or JavaScript. If significant bits overflow, they are simply gone. Always verify that your value has enough headroom in the data type before shifting. In C#, you can use a `checked` context to detect arithmetic overflow, but this does not apply to shift operators — shifts are always unchecked.
```

---

## Zero-Fill on the Right

Regardless of the language, the sign of the value, or the data type, left shift **always** fills vacated positions on the right with `0`. This is consistent across C++, C#, and JavaScript.

This is in contrast to the [[Right Shift]] operator, where the fill behavior depends on whether the value is signed (arithmetic shift, fills with sign bit) or unsigned (logical shift, fills with `0`). See also [[Unsigned Right Shift]] for JavaScript's `>>>` operator.

---

## Language-Specific Behavior

### C++

```cpp
#include <cstdint>

int main() {
    int a = 5;           // 0b00000101
    int b = a << 3;      // 0b00101000 = 40
    
    uint8_t mask = 1 << 4;   // 0b00010000 = 16
    
    // Fast multiply by 8
    int value = 12;
    int result = value << 3; // 12 * 8 = 96
    
    return 0;
}
```

```ad-warning
title: Undefined Behavior in C++
C++ has strict rules about what constitutes valid left shift usage. Violating them causes **undefined behavior** (UB) — the compiler may produce any result, including crashes, wrong values, or code that appears to work but fails unpredictably.

**UB Case 1 — Shifting negative values:**
```cpp
int x = -1;
int y = x << 2;  // UNDEFINED BEHAVIOR (until C++20)
```
In C++20, left shift of negative values was made well-defined (using two's complement), but prior standards treat it as UB.

**UB Case 2 — Shift count >= bit width:**
```cpp
int x = 1;
int y = x << 32;  // UB: int is 32 bits, shift count must be < 32
int z = x << -1;  // UB: negative shift count
```
The shift count must be in the range `[0, bit_width - 1]`. Shifting by exactly the bit width or more is undefined.
```

### C#

```csharp
int a = 5;           // 0b00000101
int b = a << 3;      // 0b00101000 = 40

uint mask = 1u << 4; // 0b00010000 = 16

// Packing two 16-bit values into one 32-bit int
int packed = (highValue << 16) | lowValue;

long bigShift = 1L << 40; // Valid: long is 64-bit
```

```ad-note
title: C# Shift Count Masking
C# does NOT cause undefined behavior with large shift counts. Instead, it **masks** the shift count:
- For `int` (32-bit): the count is masked with `& 0x1F` (keeps only the low 5 bits, range 0-31)
- For `long` (64-bit): the count is masked with `& 0x3F` (keeps only the low 6 bits, range 0-63)

This means:
```csharp
int x = 1;
int a = x << 32;   // Equivalent to x << 0  (32 & 0x1F = 0) → result is 1
int b = x << 33;   // Equivalent to x << 1  (33 & 0x1F = 1) → result is 2
int c = x << 64;   // Equivalent to x << 0  (64 & 0x1F = 0) → result is 1

long y = 1L;
long d = y << 64;  // Equivalent to y << 0  (64 & 0x3F = 0) → result is 1
```
This is well-defined but potentially surprising — shifting an `int` by 32 does NOT produce zero.
```

### JavaScript

```javascript
let a = 5;            // 0b00000101
let b = a << 3;       // 0b00101000 = 40

let mask = 1 << 4;    // 0b00010000 = 16

// Creating flags
const READ    = 1 << 0;  // 1
const WRITE   = 1 << 1;  // 2
const EXECUTE = 1 << 2;  // 4
```

```ad-warning
title: JavaScript 32-bit Conversion
JavaScript converts both operands to **signed 32-bit integers** before performing any bitwise operation, including left shift. This means:
- Numbers larger than 2^31 - 1 are truncated
- Floating-point numbers are floored to integers
- The result is always a signed 32-bit integer

```javascript
// Large numbers are truncated to 32 bits
let x = 0xFFFFFFFF;   // 4294967295 (larger than 32-bit signed)
let y = x << 1;       // -2  (not what you might expect)

// 1 << 31 gives a negative number (sets the sign bit)
let z = 1 << 31;      // -2147483648

// Floats are truncated
let w = 5.9 << 1;     // 10  (5.9 is floored to 5, then shifted)
```
For values that need more than 31 bits of precision, use BigInt: `1n << 40n`.
```

---

## Common Use Cases

### 1. Creating Bit Masks

The expression `1 << n` creates a mask with only the nth bit set. This is the foundation of [[Creating Bit Masks|bit mask creation]] and is used extensively with [[AND Operator|AND (`&`)]] to test bits and [[OR Operator|OR (`|`)]] to set bits. See [[Check Set and Clear Bits]] for complete patterns.

```cpp
// Create a mask for bit 5
int mask = 1 << 5;   // 0b00100000 = 32

// Set bit 5
flags |= (1 << 5);

// Clear bit 5
flags &= ~(1 << 5);

// Test bit 5
if (flags & (1 << 5)) { /* bit is set */ }
```

### 2. Fast Multiplication by Powers of 2

When performance matters and the multiplier is a known power of 2, left shift communicates intent clearly in systems-level code.

```csharp
int kilobytes = 4;
int bytes = kilobytes << 10;   // 4 * 1024 = 4096

int megabytes = 2;
int bytesInMB = megabytes << 20; // 2 * 1048576 = 2097152
```

### 3. Packing Values into Bit Fields

Left shift is essential for combining multiple smaller values into a single larger integer. This is common in graphics programming, network protocols, and hardware registers.

```csharp
// Pack RGBA color into a single 32-bit integer
byte r = 255, g = 128, b = 64, a = 200;
uint color = ((uint)a << 24) | ((uint)r << 16) | ((uint)g << 8) | b;
// color = 0xC8FF8040

// Pack two 16-bit values into one 32-bit value
ushort x = 1920, y = 1080;
uint packed = ((uint)x << 16) | y;
```

```ad-tip
title: Combining Left Shift with Other Operators
Left shift rarely works alone. A typical bit-packing pattern uses `<<` to position a value, `|` ([[OR Operator]]) to combine values, `&` ([[AND Operator]]) to extract values, and `>>` ([[Right Shift]]) to unpack them. Master these four together.
```

---

## Relationship to Other Shift Operators

| Operator | Symbol | Fill Behavior | Available In |
|---|---|---|---|
| Left Shift | `<<` | Always fills with 0 | C++, C#, JS |
| [[Right Shift]] | `>>` | Sign bit (signed) or 0 (unsigned) | C++, C#, JS |
| [[Unsigned Right Shift]] | `>>>` | Always fills with 0 | JS, C# (since C# 11) |

Left shift does not have a "signed" vs "unsigned" variant because it always fills with zeros on the right. The distinction only matters for [[Right Shift]], where the fill bit depends on whether the value is a [[Signed and Unsigned Integers|signed or unsigned]] type.

---

## Quick Reference

```
Value:   v << n

Effect:  Shifts all bits in v left by n positions
         High bits are discarded
         Low bits are zero-filled
         Equivalent to v * 2^n (when no overflow)

C++:     Negative v or n >= bit_width is UNDEFINED BEHAVIOR
C#:      n is masked: n & 0x1F (int), n & 0x3F (long)
JS:      Operands converted to signed 32-bit int first

Common patterns:
  1 << n            Create mask for bit n
  v << n            Multiply v by 2^n
  (v << offset) | packed   Pack v into a bit field
```

---

## Related Topics

- [[Right Shift]]
- [[Unsigned Right Shift]]
- [[Creating Bit Masks]]
- [[Binary Number System]]
- [[Check Set and Clear Bits]]
- [[AND Operator]]
- [[OR Operator]]
- [[Signed and Unsigned Integers]]
- [[Bits Bytes and Words]]
