---
tags:
  - bit-manipulation
  - signed
  - unsigned
  - integer-representation
---

## The Core Distinction

The same bit pattern can represent different values depending on whether you interpret it as **signed** (can be negative) or **unsigned** (always non-negative).

```
Bit pattern:  1111 1110

Unsigned:     254
Signed:       -2  (two's complement)
```

The hardware doesn't know which interpretation is "correct" — that's determined by the data type in your code.

---

## Unsigned Integer Representation

Unsigned integers are straightforward: the binary value directly equals the decimal value using standard [[Binary Number System|positional notation]].

### Range for N-bit Unsigned

```
Minimum: 0
Maximum: 2^N - 1
```

| Bits | Min | Max                        |
|------|-----|----------------------------|
| 8    | 0   | 255                         |
| 16   | 0   | 65,535                      |
| 32   | 0   | 4,294,967,295               |
| 64   | 0   | 18,446,744,073,709,551,615  |

```
8-bit unsigned examples:

0000 0000 =   0  (minimum)
0000 0001 =   1
0111 1111 = 127
1000 0000 = 128
1111 1110 = 254
1111 1111 = 255  (maximum)
```

---

## Signed Integer Representations

There are three historical approaches to representing negative numbers in binary. Only **two's complement** is used in modern hardware.

### 1. Sign-Magnitude (Historical)

The **most significant bit (MSB)** is a sign flag: `0` = positive, `1` = negative. The remaining bits hold the magnitude.

```
+5 = 0 0000101
-5 = 1 0000101
     ^
     sign bit
```

**Problems with sign-magnitude:**
- Two representations of zero: `0 0000000` (+0) and `1 0000000` (-0)
- Addition/subtraction circuits are more complex (must check the sign bit first)
- Range: -(2^(N-1) - 1) to +(2^(N-1) - 1), with a wasted bit pattern

### 2. Ones' Complement (Historical)

Negative numbers are formed by flipping all bits of the positive number.

```
+5 = 0000 0101
-5 = 1111 1010  (each bit flipped)
```

**Problems with ones' complement:**
- Still has two zeros: `0000 0000` (+0) and `1111 1111` (-0)
- End-around carry required for addition

### 3. Two's Complement (The Standard)

Negative numbers are formed by flipping all bits and adding 1. This is the representation used by virtually all modern hardware.

```
+5 = 0000 0101
~5 = 1111 1010  (flip bits)
-5 = 1111 1011  (add 1)
```

See [[Twos Complement]] for a deep dive into why two's complement won and how it works.

---

## Range Comparison: Signed vs Unsigned (N bits)

| Type     | Minimum          | Maximum          | Total Values |
|----------|------------------|------------------|-------------|
| Unsigned | 0                | 2^N - 1          | 2^N         |
| Signed   | -2^(N-1)         | 2^(N-1) - 1      | 2^N         |

Both represent exactly **2^N** distinct values. Signed just shifts half the range to the negative side.

### 8-bit Example

```
Unsigned:  0 ............... 255
           |                 |
Signed:   -128 ..... 0 ..... 127
           |         |       |

Both have 256 possible values.
```

### Visual: 8-bit Signed vs Unsigned Number Line

```
Bit pattern  Unsigned  Signed(2's comp)
-----------  --------  ----------------
0000 0000       0            0
0000 0001       1            1
0000 0010       2            2
  ...         ...          ...
0111 1110     126          126
0111 1111     127          127     <-- same up to here
1000 0000     128         -128     <-- interpretations diverge!
1000 0001     129         -127
  ...         ...          ...
1111 1110     254           -2
1111 1111     255           -1
```

```ad-tip
title: When the MSB is 1
For unsigned, a `1` in the MSB contributes its positional value (128 for 8-bit). For signed two's complement, a `1` in the MSB means the number is negative. This is why `0111 1111` (127) and `1000 0000` (128 unsigned / -128 signed) are the critical boundary.
```

---

## Language-Specific Handling

### C++

C++ provides both signed and unsigned variants for all integer types:

```cpp
#include <iostream>
#include <cstdint>
#include <limits>

int main() {
    // Signed types (default)
    int          si = -42;
    int8_t       s8 = -100;
    int16_t     s16 = -30000;
    int32_t     s32 = -2000000000;
    int64_t     s64 = -9000000000000000000LL;

    // Unsigned types
    unsigned int  ui = 42;
    uint8_t       u8 = 200;
    uint16_t     u16 = 60000;
    uint32_t     u32 = 4000000000U;
    uint64_t     u64 = 18000000000000000000ULL;

    // Ranges
    std::cout << "int8_t:   " << (int)std::numeric_limits<int8_t>::min()
              << " to " << (int)std::numeric_limits<int8_t>::max() << std::endl;
    // -128 to 127

    std::cout << "uint8_t:  " << (int)std::numeric_limits<uint8_t>::min()
              << " to " << (int)std::numeric_limits<uint8_t>::max() << std::endl;
    // 0 to 255

    std::cout << "int32_t:  " << std::numeric_limits<int32_t>::min()
              << " to " << std::numeric_limits<int32_t>::max() << std::endl;
    // -2147483648 to 2147483647

    std::cout << "uint32_t: " << std::numeric_limits<uint32_t>::min()
              << " to " << std::numeric_limits<uint32_t>::max() << std::endl;
    // 0 to 4294967295

    return 0;
}
```

```ad-warning
title: Signed/unsigned comparison in C++
Comparing a signed value to an unsigned value causes **implicit conversion** to unsigned, which can produce shocking results:

`int s = -1; unsigned int u = 1;`
`if (s < u)` evaluates to **false** because `-1` becomes `4294967295` when converted to `unsigned int`. The compiler warns about this — never ignore `-Wsign-compare`.
```

### C#

C# has explicit signed and unsigned types with fixed sizes:

```csharp
using System;

class Program
{
    static void Main()
    {
        // Signed types
        sbyte  s8  = -100;        // 8-bit signed
        short  s16 = -30000;      // 16-bit signed
        int    s32 = -2000000000; // 32-bit signed
        long   s64 = -9000000000000000000L; // 64-bit signed

        // Unsigned types
        byte   u8  = 200;         // 8-bit unsigned
        ushort u16 = 60000;       // 16-bit unsigned
        uint   u32 = 4000000000U; // 32-bit unsigned
        ulong  u64 = 18000000000000000000UL; // 64-bit unsigned

        // Ranges
        Console.WriteLine($"sbyte:  {sbyte.MinValue} to {sbyte.MaxValue}");    // -128 to 127
        Console.WriteLine($"byte:   {byte.MinValue} to {byte.MaxValue}");      // 0 to 255
        Console.WriteLine($"int:    {int.MinValue} to {int.MaxValue}");        // -2147483648 to 2147483647
        Console.WriteLine($"uint:   {uint.MinValue} to {uint.MaxValue}");      // 0 to 4294967295

        // Casting between signed and unsigned
        int signed_val = -1;
        uint unsigned_val = (uint)signed_val;  // unchecked: 4294967295
        Console.WriteLine($"-1 as uint: {unsigned_val}");

        // Checked context catches overflow
        try
        {
            checked
            {
                uint safe = (uint)signed_val;  // Throws OverflowException
            }
        }
        catch (OverflowException)
        {
            Console.WriteLine("Overflow detected in checked context!");
        }
    }
}
```

```ad-tip
title: CLS compliance
C#'s unsigned types (`uint`, `ulong`, `ushort`, `sbyte`) are NOT CLS-compliant. If you're writing a public API intended for cross-language consumption in .NET, prefer signed types for public members.
```

### JavaScript

JavaScript's `number` type is always a 64-bit IEEE 754 double. It has **no unsigned integer type**. However, bitwise operations use 32-bit signed integers internally.

```javascript
// JavaScript has no unsigned integer type
let x = -1;
console.log(x);  // -1

// Bitwise operations use 32-bit signed integers
console.log(x | 0);       // -1  (forces to 32-bit signed int)
console.log(x >>> 0);     // 4294967295  (unsigned right shift treats as unsigned)

// The >>> operator is the ONLY way to treat a number as unsigned in bitwise context
// See [[Unsigned Right Shift]] for details

// Demonstrating the 32-bit signed limit
console.log(0x7FFFFFFF);       // 2147483647  (max 32-bit signed)
console.log(0x7FFFFFFF | 0);   // 2147483647
console.log(0x80000000 | 0);   // -2147483648 (interpreted as signed!)
console.log(0x80000000 >>> 0); // 2147483648  (treated as unsigned)

// TypedArrays for explicit unsigned integers
let u8  = new Uint8Array([255]);     // 8-bit unsigned
let u16 = new Uint16Array([65535]);  // 16-bit unsigned
let u32 = new Uint32Array([4294967295]); // 32-bit unsigned
let i8  = new Int8Array([255]);      // 8-bit signed: wraps to -1!

console.log(u8[0]);   // 255
console.log(i8[0]);   // -1  (same bit pattern, different interpretation)
```

```ad-warning
title: JavaScript bitwise gotcha
Every bitwise operator except `>>>` converts its operands to **32-bit signed** integers. This means:
- Values above 2^31 - 1 become negative after `|`, `&`, `^`, `~`, `<<`, `>>`
- Only `>>>` (unsigned [[Right Shift|right shift]]) interprets the result as unsigned
- If you need true unsigned 32-bit arithmetic, always use `>>> 0` to convert back
```

---

## Overflow Behavior

### Unsigned Overflow (Wrapping)

When an unsigned value exceeds its maximum, it wraps around to 0:

```
8-bit unsigned:

  255 + 1 = 0      (wraps around)
  255 + 2 = 1
  200 + 100 = 44   (300 mod 256 = 44)

Binary view:
  1111 1111  (255)
+ 0000 0001  (  1)
-----------
1 0000 0000  (the carry is lost, result = 0)
```

### Signed Overflow (Undefined/Wrapping)

When a signed value exceeds its range:

```
8-bit signed two's complement:

  127 + 1 = -128   (wraps to most negative)
  127 + 2 = -127

Binary view:
  0111 1111  (127)
+ 0000 0001  (  1)
-----------
  1000 0000  (-128 in signed interpretation)
```

```ad-warning
title: Signed overflow is undefined behavior in C/C++
In C and C++, signed integer overflow is **undefined behavior** (UB). The compiler may assume it never happens and optimize accordingly. This can cause surprising bugs. Unsigned overflow, by contrast, is well-defined wrapping behavior.

In C#, overflow is defined (wrapping) in unchecked context and throws `OverflowException` in `checked` context. In JavaScript, numbers silently lose precision past `Number.MAX_SAFE_INTEGER`.
```

---

## When to Use Signed vs Unsigned

| Use Case                        | Recommendation | Why                                           |
|---------------------------------|---------------|-----------------------------------------------|
| General-purpose integers        | Signed        | Avoid implicit conversion bugs                |
| Loop counters                   | Signed        | Prevent underflow with decrementing loops      |
| Bit manipulation / flags        | Unsigned      | Behavior of [[Right Shift]] is predictable    |
| Byte buffers / raw data         | Unsigned      | Bytes are naturally 0-255                      |
| Array sizes / counts            | Unsigned (C++), Signed (C#/JS) | Language convention |
| Network protocol fields         | Unsigned      | Protocols define unsigned fields               |

```ad-tip
title: Google C++ Style Guide recommendation
"Use unsigned types only for bit manipulation. Use signed types for all other integer work, including loop variables and sizes." This avoids the many subtle bugs that arise from signed/unsigned implicit conversions.
```

---

## Related Notes

- [[Twos Complement]] — deep dive into the standard signed representation
- [[Bits Bytes and Words]] — data sizes and ranges
- [[Binary Number System]] — foundational binary concepts
- [[Right Shift]] — arithmetic vs logical right shift depends on signed/unsigned
- [[NOT Operator]] — bitwise NOT interacts differently with signed values
