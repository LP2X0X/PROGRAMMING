---
tags:
  - bit-manipulation
  - twos-complement
  - integer-representation
---

## What Is Two's Complement?

Two's complement is the standard method used by virtually all modern CPUs to represent **signed integers** in binary. It assigns the most significant bit (MSB) a negative weight, creating a system where:

- Positive numbers look the same as unsigned binary
- Negative numbers are formed by inverting all bits and adding 1
- There is exactly **one representation of zero**
- Addition of positive and negative numbers works with the same hardware circuit as unsigned addition

---

## Why Two's Complement Won

Three representations were historically considered (see [[Signed and Unsigned Integers]] for details):

| Property                    | Sign-Magnitude | Ones' Complement | Two's Complement |
|-----------------------------|:--------------:|:----------------:|:----------------:|
| Single representation of 0  | No (-0 and +0) | No (-0 and +0)   | **Yes**          |
| Simple addition circuit      | No              | Partially         | **Yes**          |
| Simple negation              | Yes (flip MSB)  | Yes (flip all)    | Flip all + add 1 |
| Symmetric range              | Yes             | Yes               | No (one extra negative) |
| Used in modern hardware      | No              | No                | **Yes**          |

The killer advantage: **the same adder circuit works for both signed and unsigned addition**. No special hardware needed.

---

## How to Compute Two's Complement (Negation)

To negate a number (find `-x` from `x`):

**Step 1:** Invert all bits (apply [[NOT Operator|bitwise NOT]])
**Step 2:** Add 1

### Example: Negate +5 to get -5 (8-bit)

```
Start:    0000 0101   ( +5 )
Step 1:   1111 1010   ( invert all bits )
Step 2:   1111 1011   ( add 1 = -5 )
```

### Verify: Negate -5 back to +5

```
Start:    1111 1011   ( -5 )
Step 1:   0000 0100   ( invert all bits )
Step 2:   0000 0101   ( add 1 = +5 )
```

It works both ways.

### Shortcut Method: Find Rightmost 1, Flip Everything Left of It

```
Original: 0101 1000   ( +88 )
                   ^---- rightmost 1 is at bit 3
Keep bits 0-3:     1000
Flip bits 4-7: 1010
Result:   1010 1000   ( -88 )
```

---

## How Two's Complement Encodes Values

In an N-bit two's complement number, the MSB has a **negative weight** of -2^(N-1):

```
For 8 bits, the bit weights are:

Bit:      7      6     5     4     3     2     1     0
Weight: -128    64    32    16     8     4     2     1
```

### Example: Interpret `1110 0011`

```
Bit:      1      1     1     0     0     0     1     1
Weight: -128    64    32    16     8     4     2     1

Value = -128 + 64 + 32 + 0 + 0 + 0 + 2 + 1
      = -128 + 99
      = -29
```

### Example: Interpret `0110 0100`

```
Bit:      0      1     1     0     0     1     0     0
Weight: -128    64    32    16     8     4     2     1

Value = 0 + 64 + 32 + 0 + 0 + 4 + 0 + 0
      = 100
```

---

## The Single Zero Advantage

In sign-magnitude and ones' complement, zero has two representations:

```
Sign-magnitude:    0000 0000 (+0)  and  1000 0000 (-0)
Ones' complement:  0000 0000 (+0)  and  1111 1111 (-0)
```

In two's complement, only one pattern represents zero:

```
Two's complement:  0000 0000 (0)

What about 1111 1111?
Weight: -128 + 64 + 32 + 16 + 8 + 4 + 2 + 1 = -1  (not zero!)
```

This eliminates edge cases in comparison logic: `x == 0` is just "are all bits zero?"

---

## Addition Just Works

The beauty of two's complement is that **addition uses the same binary adder regardless of sign**.

### Example: 5 + (-3) = 2

```
    0000 0101   ( +5 )
  + 1111 1101   ( -3 )
  -----------
  1 0000 0010   ( +2, carry out is discarded )
```

### Example: -5 + (-3) = -8

```
    1111 1011   ( -5 )
  + 1111 1101   ( -3 )
  -----------
  1 1111 1000   ( -8, carry out is discarded )
```

### Example: 10 + (-10) = 0

```
    0000 1010   ( +10 )
  + 1111 0110   ( -10 )
  -----------
  1 0000 0000   ( 0, carry out is discarded )
```

```ad-tip
title: Why the carry is discarded
In an N-bit system, the carry out of bit N-1 is simply dropped. This is equivalent to arithmetic modulo 2^N, which is exactly what makes two's complement work. The hardware doesn't need to know whether the numbers are signed or unsigned — the bit-level addition is identical.
```

---

## Overflow Detection

Overflow occurs when the result of an addition/subtraction cannot be represented in the available bits. For two's complement, overflow happens when:

**Adding two positives produces a negative, or adding two negatives produces a positive.**

Equivalently: overflow occurs when the **carry into the MSB differs from the carry out of the MSB**.

### Example: Overflow! 100 + 50 = -106 (?!)

```
    0110 0100   ( +100 )
  + 0011 0010   ( +50  )
  -----------
    1001 0110   ( -106 in signed interpretation! )

Both operands are positive, but the result looks negative.
OVERFLOW detected!
```

### Example: No overflow. -100 + 50 = -50

```
    1001 1100   ( -100 )
  + 0011 0010   (  +50 )
  -----------
    1100 1110   (  -50  )

One positive, one negative — overflow is impossible.
```

```ad-warning
title: Overflow vs Carry
**Carry** (C flag): a bit carried out of the MSB. Relevant for unsigned arithmetic.
**Overflow** (V flag): result has wrong sign. Relevant for signed arithmetic.

They are independent — you can have carry without overflow, overflow without carry, both, or neither.
```

---

## The "Weird" Minimum Value

Every N-bit two's complement system has an **asymmetric range**:

```
8-bit:   -128  to  +127     (not -127 to +127)
16-bit:  -32768 to +32767
32-bit:  -2,147,483,648 to +2,147,483,647
```

The minimum value has **no positive counterpart** that fits in N bits:

```
-128 = 1000 0000

Negate it:
  Invert: 0111 1111
  Add 1:  1000 0000  <-- we get -128 again!
```

```ad-warning
title: Negating the minimum value
In C/C++, writing `-(-128)` for an `int8_t` is **undefined behavior** because `+128` cannot be represented in 8-bit signed two's complement. In C#, it throws `OverflowException` in a `checked` context. In JavaScript, this isn't an issue because numbers are 64-bit doubles.
```

This happens because the range is [-2^(N-1), 2^(N-1)-1] — there's one more negative number than positive. The extra negative value is the pattern `1000...0` (only the MSB set).

---

## 8-Bit Two's Complement Number Circle

```
                    0 (0000 0000)
                   / \
              -1 /     \ +1
        (1111 1111)  (0000 0001)
              /             \
         -2 /               \ +2
   (1111 1110)            (0000 0010)
          |                    |
          |                    |
    ...   |                    |   ...
          |                    |
         -64                 +64
   (1100 0000)            (0100 0000)
           \                 /
       -127 \             / +126
   (1000 0001)        (0111 1110)
              \         /
         -128  \     / +127
        (1000 0000) (0111 1111)

Moving clockwise:  incrementing (+1)
Moving counterclockwise: decrementing (-1)

Adding 1 to +127 wraps to -128 (overflow!)
Subtracting 1 from -128 wraps to +127 (overflow!)
```

```ad-note
title: The circle is the modular arithmetic
Two's complement integers form a ring of 2^N values. Addition wraps around the circle. Overflow occurs only when you cross the boundary between +127 and -128 (for 8-bit).
```

---

## Subtraction via Addition

Two's complement makes subtraction trivial:

```
A - B = A + (-B) = A + (~B + 1)
```

Hardware implements subtraction by:
1. Inverting all bits of B (using [[NOT Operator|NOT gates]])
2. Setting the carry-in of the adder to 1 (adds the +1)
3. Adding normally

No separate subtraction circuit needed.

---

## Sign Extension

When converting a signed value to a wider type, the **sign bit is replicated** to fill the new bits. This preserves the numerical value.

```
8-bit to 16-bit:

+5:    0000 0101  -->  0000 0000 0000 0101   (fill with 0s)
-5:    1111 1011  -->  1111 1111 1111 1011   (fill with 1s)

Verify -5 in 16 bits:
-32768 + 16384 + ... + 8 + 2 + 1
= sum of all positive weights minus 32768
= (32768 - 1) - 32768 + (the original 8-bit value)
= -5  (correct!)
```

```ad-tip
title: Zero extension vs sign extension
**Sign extension** (for signed types): copy the MSB into all new upper bits.
**Zero extension** (for unsigned types): fill new upper bits with `0`.

This is why [[Right Shift]] has two variants: **arithmetic shift** (sign-extends) and **logical shift** (zero-extends).
```

---

## Code Examples

### C++

```cpp
#include <iostream>
#include <bitset>
#include <cstdint>
#include <climits>

int main() {
    // Two's complement negation
    int8_t x = 5;
    int8_t neg_x = ~x + 1;  // Manual two's complement
    std::cout << "x = " << (int)x << ", -x = " << (int)neg_x << std::endl;
    // x = 5, -x = -5

    // Verify: compiler negation matches manual two's complement
    std::cout << "Compiler -x: " << (int)(-x) << std::endl;  // -5

    // Show bit patterns
    std::cout << "+5:  " << std::bitset<8>(x) << std::endl;     // 00000101
    std::cout << "~5:  " << std::bitset<8>(~x) << std::endl;    // 11111010
    std::cout << "-5:  " << std::bitset<8>(neg_x) << std::endl; // 11111011

    // The asymmetric range
    std::cout << "INT8_MIN: "  << (int)INT8_MIN  << std::endl;  // -128
    std::cout << "INT8_MAX: "  << (int)INT8_MAX  << std::endl;  // 127
    std::cout << "-INT8_MIN: undefined behavior!" << std::endl;

    // Sign extension happens automatically
    int8_t  small = -5;           // 1111 1011
    int32_t big   = small;        // 1111...1111 1011 (sign-extended)
    std::cout << "int8 -5 as int32: " << big << std::endl;  // -5

    // Casting between signed and unsigned (reinterpret bits)
    int8_t  signed_val = -1;                  // 1111 1111
    uint8_t unsigned_val = (uint8_t)signed_val; // 1111 1111 = 255
    std::cout << "Signed -1 as unsigned: " << (int)unsigned_val << std::endl;  // 255

    return 0;
}
```

### C#

```csharp
using System;

class Program
{
    static void Main()
    {
        // Two's complement negation
        sbyte x = 5;
        sbyte negX = (sbyte)(~x + 1);  // Manual two's complement
        Console.WriteLine($"x = {x}, -x = {negX}");  // x = 5, -x = -5

        // Show bit patterns
        Console.WriteLine($"+5:  {Convert.ToString((byte)x, 2).PadLeft(8, '0')}");       // 00000101
        Console.WriteLine($"~5:  {Convert.ToString((byte)(~x & 0xFF), 2).PadLeft(8, '0')}"); // 11111010
        Console.WriteLine($"-5:  {Convert.ToString((byte)negX, 2).PadLeft(8, '0')}");     // 11111011

        // The asymmetric range
        Console.WriteLine($"sbyte min: {sbyte.MinValue}");   // -128
        Console.WriteLine($"sbyte max: {sbyte.MaxValue}");   // 127
        Console.WriteLine($"int min:   {int.MinValue}");     // -2147483648
        Console.WriteLine($"int max:   {int.MaxValue}");     // 2147483647

        // Checked negation of MinValue throws
        try
        {
            checked
            {
                int minVal = int.MinValue;
                int negMin = -minVal;  // OverflowException!
            }
        }
        catch (OverflowException)
        {
            Console.WriteLine("Cannot negate int.MinValue in checked context!");
        }

        // Sign extension is automatic
        sbyte small = -5;
        int big = small;  // Sign-extended to 32 bits
        Console.WriteLine($"sbyte -5 as int: {big}");  // -5

        // Reinterpret bits: signed <-> unsigned
        sbyte signedVal = -1;                          // 1111 1111
        byte unsignedVal = unchecked((byte)signedVal); // 1111 1111 = 255
        Console.WriteLine($"Signed -1 as unsigned: {unsignedVal}");  // 255
    }
}
```

### JavaScript

```javascript
// Two's complement negation (32-bit)
let x = 5;
let negX = (~x + 1);  // Manual two's complement
console.log(`x = ${x}, -x = ${negX}`);  // x = 5, -x = -5

// This works because ~ is bitwise NOT on 32-bit signed int
// ~5 = -6 (in two's complement), then -6 + 1 = -5

// Show bit patterns (32-bit)
function toBin32(n) {
    return (n >>> 0).toString(2).padStart(32, '0');
}

console.log(`+5:  ${toBin32(5)}`);
// 00000000000000000000000000000101
console.log(`~5:  ${toBin32(~5)}`);
// 11111111111111111111111111111010
console.log(`-5:  ${toBin32(-5)}`);
// 11111111111111111111111111111011

// The 32-bit signed range in bitwise operations
console.log(`Max 32-bit signed: ${0x7FFFFFFF}`);        // 2147483647
console.log(`Min 32-bit signed: ${(0x80000000 | 0)}`);  // -2147483648

// JavaScript numbers are 64-bit doubles, so no true overflow
// for regular arithmetic (until you exceed Number.MAX_SAFE_INTEGER)
let big = 2147483647 + 1;
console.log(`2^31 - 1 + 1 = ${big}`);  // 2147483648 (no overflow in JS math)

// But bitwise operations wrap around at 32 bits
console.log(`(2^31 - 1 + 1) | 0 = ${(big | 0)}`);  // -2147483648 (overflow!)

// Two's complement is visible with TypedArrays
let arr = new Int8Array([200]);   // 200 = 1100 1000, interpreted as signed
console.log(arr[0]);              // -56 (two's complement interpretation)

let uarr = new Uint8Array([200]);
console.log(uarr[0]);             // 200 (unsigned interpretation)
```

---

## Summary Table

| Concept | Formula / Rule |
|---------|---------------|
| Negate x | `~x + 1` or invert bits, add 1 |
| N-bit range | -2^(N-1) to 2^(N-1) - 1 |
| MSB weight | -2^(N-1) (negative!) |
| Check if negative | MSB == 1 |
| Weird minimum | -2^(N-1) has no positive counterpart in N bits |
| Subtraction | `A - B = A + (~B + 1)` |
| Sign extension | Copy MSB into new upper bits |

---

## Related Notes

- [[Signed and Unsigned Integers]] — overview of all signed representations
- [[NOT Operator]] — the bitwise NOT used in negation
- [[Binary Number System]] — positional notation fundamentals
- [[Check Set and Clear Bits]] — practical bit manipulation using two's complement properties
