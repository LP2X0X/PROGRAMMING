---
tags:
  - bit-manipulation
  - cpp
  - language-specific
---

## Overview

C++ provides extensive facilities for bit manipulation, from raw bitwise operators to the modern C++20 `<bit>` header. However, C++ also has more pitfalls than most languages -- undefined behavior lurks in many shifting and overflow scenarios. This note covers every C++-specific feature and gotcha related to working with bits.

---

## Integer Types and Sizes

### Fundamental Types (Platform-Dependent Widths)

C++ fundamental integer types do **not** have guaranteed sizes. The standard only specifies minimum ranges:

| Type          | Minimum Bits | Typical (x86-64) | Guaranteed Minimum Range               |
|---------------|-------------|-------------------|-----------------------------------------|
| `char`        | 8           | 8                 | -127 to 127 or 0 to 255                |
| `short`       | 16          | 16                | -32,767 to 32,767                       |
| `int`         | 16          | 32                | -32,767 to 32,767                       |
| `long`        | 32          | 32 (Win), 64 (Linux) | -2,147,483,647 to 2,147,483,647     |
| `long long`   | 64          | 64                | -9.2 x 10^18 to 9.2 x 10^18           |

```ad-warning
title: int is NOT always 32 bits
On most modern desktop platforms, `int` is 32 bits, but the C++ standard only guarantees at least 16 bits. On embedded systems or older architectures, `int` can be 16 bits. Never assume a specific size for fundamental types in portable bit manipulation code.
```

### Fixed-Width Types (`<cstdint>`)

For bit manipulation, always prefer fixed-width types from `<cstdint>`:

```cpp
#include <cstdint>

int8_t    a;  // Exactly 8 bits, signed
int16_t   b;  // Exactly 16 bits, signed
int32_t   c;  // Exactly 32 bits, signed
int64_t   d;  // Exactly 64 bits, signed

uint8_t   e;  // Exactly 8 bits, unsigned
uint16_t  f;  // Exactly 16 bits, unsigned
uint32_t  g;  // Exactly 32 bits, unsigned
uint64_t  h;  // Exactly 64 bits, unsigned
```

| Type       | Signed? | Bits | Min Value                    | Max Value                     |
|------------|---------|------|------------------------------|-------------------------------|
| `int8_t`   | Yes     | 8    | -128                         | 127                           |
| `uint8_t`  | No      | 8    | 0                            | 255                           |
| `int16_t`  | Yes     | 16   | -32,768                      | 32,767                        |
| `uint16_t` | No      | 16   | 0                            | 65,535                        |
| `int32_t`  | Yes     | 32   | -2,147,483,648               | 2,147,483,647                 |
| `uint32_t` | No      | 32   | 0                            | 4,294,967,295                 |
| `int64_t`  | Yes     | 64   | -9,223,372,036,854,775,808   | 9,223,372,036,854,775,807     |
| `uint64_t` | No      | 64   | 0                            | 18,446,744,073,709,551,615    |

```ad-tip
title: Always use fixed-width types for bit manipulation
When writing bit manipulation code, always use `uint32_t`, `uint64_t`, etc. from `<cstdint>`. This makes your code portable and your bit positions predictable. See [[Bits Bytes and Words]] for data unit sizes and [[Signed and Unsigned Integers]] for signed vs unsigned representation.
```

### Additional `<cstdint>` Types

```cpp
// "Least" types: at least N bits, might be larger
uint_least8_t   x1;  // At least 8 bits unsigned
uint_least32_t  x2;  // At least 32 bits unsigned

// "Fast" types: at least N bits, fastest type that size or larger
uint_fast8_t    y1;  // Fastest type with at least 8 bits
uint_fast32_t   y2;  // Fastest type with at least 32 bits

// Maximum-width types
intmax_t   z1;  // Largest supported signed integer type
uintmax_t  z2;  // Largest supported unsigned integer type

// Pointer-sized types
intptr_t   p1;  // Signed integer that can hold a pointer
uintptr_t  p2;  // Unsigned integer that can hold a pointer
size_t     s;   // Unsigned type for sizes and counts
```

---

## Binary and Hexadecimal Literals

### Binary Literals (`0b` prefix, C++14)

```cpp
int flags = 0b1010'1100;      // 172 in decimal
uint8_t mask = 0b0000'1111;   // Lower nibble mask
uint32_t pattern = 0b1100'0011'1010'0101'0000'1111'1111'0000;
```

### Hexadecimal Literals (`0x` prefix)

```cpp
int color = 0xFF'00'FF;       // Magenta (RGB)
uint32_t addr = 0xDEAD'BEEF;  // Classic debug pattern
uint8_t high = 0xF0;          // Upper nibble mask
```

### Digit Separators (`'`, C++14)

C++ uses the single quote `'` as a digit separator (unlike C# which uses `_`):

```cpp
// Group binary by nibble
uint16_t grouped = 0b1010'0011'1100'0101;

// Group hex by byte
uint32_t color = 0xFF'A0'B0'C0;

// Decimal grouping
int million = 1'000'000;

// Octal grouping
int octal = 0'177'777;
```

```ad-note
title: C++ vs C# digit separator
C++ uses `'` (single quote), C# uses `_` (underscore). A common mistake when switching between languages.
```

---

## Bitwise Operators

C++ supports the same fundamental bitwise operators as other C-family languages. They work on all integer types.

| Operator | Name               | Example        | Link                         |
|----------|--------------------|----------------|------------------------------|
| `&`      | Bitwise AND        | `a & b`        | [[AND Operator]]             |
| `\|`     | Bitwise OR         | `a \| b`       | [[OR Operator]]              |
| `^`      | Bitwise XOR        | `a ^ b`        | [[XOR Operator]]             |
| `~`      | Bitwise NOT        | `~a`           | [[NOT Operator]]             |
| `<<`     | Left shift         | `a << 3`       | [[Left Shift]]               |
| `>>`     | Right shift        | `a >> 3`       | [[Right Shift]]              |

### Compound Assignment Operators

```cpp
uint32_t flags = 0b1010;
flags &= 0b1100;   // flags = flags & 0b1100  --> 0b1000
flags |= 0b0011;   // flags = flags | 0b0011  --> 0b1011
flags ^= 0b0101;   // flags = flags ^ 0b0101  --> 0b1110
flags <<= 2;        // flags = flags << 2      --> 0b111000
flags >>= 1;        // flags = flags >> 1      --> 0b11100
```

### No Unsigned Right Shift Operator

```ad-warning
title: C++ has no >>> operator
Unlike C# and JavaScript, C++ has no dedicated unsigned right shift operator. The behavior of `>>` depends on the type:
- **Unsigned types**: `>>` fills with zeros (logical shift)
- **Signed types**: `>>` behavior is implementation-defined (see below)

To force a logical (zero-filling) right shift on a signed value, cast to the unsigned counterpart first. See [[Unsigned Right Shift]] for the concept.
```

```cpp
// Unsigned: >> always fills with zeros
uint32_t u = 0xFFFF'FFFF;
uint32_t uShift = u >> 4;  // 0x0FFF'FFFF (zero-filled)

// Signed: >> is implementation-defined for negative values
int32_t s = -16;  // 0xFFFF'FFF0
int32_t sShift = s >> 2;  // Usually -4 (arithmetic shift), but NOT guaranteed

// Force unsigned right shift on a signed value:
int32_t signed_val = -1;  // 0xFFFF'FFFF
uint32_t logical = static_cast<uint32_t>(signed_val) >> 4;  // 0x0FFF'FFFF
int32_t result = static_cast<int32_t>(logical);  // Store back if needed
```

### Integer Promotion Rules

When performing bitwise operations on types smaller than `int`, C++ promotes them to `int` first:

```cpp
uint8_t a = 0xFF;
uint8_t b = 0x01;

// The expression (a & b) operates on int-promoted values
// Result is int, must cast back
uint8_t result = static_cast<uint8_t>(a & b);

// Watch out for NOT on small unsigned types:
uint8_t mask = 0x0F;        // 0b0000'1111
auto notMask = ~mask;       // This is int, NOT uint8_t!
                             // Result: 0xFFFF'FF'F0 (32-bit int), not 0xF0
uint8_t correctNot = static_cast<uint8_t>(~mask);  // 0xF0
```

```ad-warning
title: The ~ trap with small unsigned types
`~` on a `uint8_t` or `uint16_t` produces an `int` result with the upper bits set. Always cast back to the intended type. For example, `~(uint8_t)0x0F` gives `0xFFFF'FFF0` (an int), not `0xF0`.
```

---

## Undefined Behavior Pitfalls

C++ has several situations where bit operations produce undefined behavior (UB). The compiler is free to do anything -- crash, produce wrong results, optimize away your code, or seemingly work fine until it doesn't.

### Shifting by Negative Amount or >= Bit Width

```ad-warning
title: UB -- Shift amount out of range
Shifting by a negative amount or by an amount >= the bit width of the type is undefined behavior.
```

```cpp
uint32_t value = 1;

// UNDEFINED BEHAVIOR:
// uint32_t bad1 = value << 32;   // shift amount == bit width
// uint32_t bad2 = value << -1;   // negative shift amount
// uint32_t bad3 = value >> 33;   // shift amount > bit width

// Safe: always validate shift amounts
int shift = 32;
uint32_t safe = (shift >= 32) ? 0 : (value << shift);

// Or use a helper:
template<typename T>
constexpr T safe_shl(T value, int shift) {
    constexpr int bits = sizeof(T) * 8;
    if (shift < 0 || shift >= bits) return 0;
    return value << shift;
}
```

### Left-Shifting a Negative Number (before C++20)

```ad-warning
title: UB before C++20 -- Left shift of negative value
Before C++20, left-shifting a negative signed integer was undefined behavior. C++20 defines that signed integers use two's complement and makes left-shifting well-defined (it shifts the bits and discards overflow, same as unsigned).
```

```cpp
int32_t neg = -1;

// Before C++20: UNDEFINED BEHAVIOR
// int32_t shifted = neg << 1;

// After C++20: Well-defined, -2
int32_t shifted = neg << 1;

// Safe alternative for pre-C++20: cast to unsigned first
uint32_t safe = static_cast<uint32_t>(neg) << 1;
int32_t result = static_cast<int32_t>(safe);
```

### Right-Shifting a Negative Number

```ad-warning
title: Implementation-defined -- Right shift of negative value
Right-shifting a negative signed integer is **implementation-defined** (not UB, but not portable). Most compilers perform an arithmetic right shift (preserving the sign bit), but the standard does not require this.
```

```cpp
int32_t neg = -8;  // 0xFFFF'FFF8

// Implementation-defined! Usually arithmetic shift:
int32_t shifted = neg >> 2;  // Typically -2 (sign bit preserved)

// If you need guaranteed logical (zero-filling) right shift:
uint32_t logical = static_cast<uint32_t>(neg) >> 2;  // 0x3FFF'FFFE
```

### Signed Integer Overflow

```ad-warning
title: UB -- Signed integer overflow
Signed integer overflow is undefined behavior in C++. Unsigned integer overflow is well-defined (wraps around modulo 2^N). For bit manipulation, prefer unsigned types to avoid UB from overflow.
```

```cpp
int32_t max = INT32_MAX;  // 2,147,483,647

// UNDEFINED BEHAVIOR:
// int32_t overflow = max + 1;

// Well-defined (unsigned wraps around):
uint32_t umax = UINT32_MAX;  // 4,294,967,295
uint32_t uwrap = umax + 1;   // 0 (well-defined wrap)
```

### Summary of UB/Implementation-Defined Behavior

| Operation                                | Status Before C++20      | Status C++20+           |
|------------------------------------------|--------------------------|--------------------------|
| Shift by negative amount                 | UB                       | UB                       |
| Shift by >= bit width                    | UB                       | UB                       |
| Left shift of negative signed integer    | UB                       | Well-defined             |
| Right shift of negative signed integer   | Implementation-defined   | Implementation-defined   |
| Signed integer overflow                  | UB                       | UB                       |
| Unsigned integer overflow (wraparound)   | Well-defined             | Well-defined             |

```ad-tip
title: Rule of thumb
For bit manipulation in C++, always prefer unsigned types (`uint32_t`, `uint64_t`, etc.). This sidesteps most UB pitfalls: unsigned overflow wraps predictably, unsigned shifts always fill with zeros, and there is no sign bit to cause implementation-defined behavior.
```

---

## The `<bit>` Header (C++20)

C++20 introduced the `<bit>` header, providing type-safe, constexpr bit manipulation functions that compile to efficient hardware instructions.

```cpp
#include <bit>
#include <cstdint>
```

### std::popcount -- Count Set Bits

Returns the number of `1` bits. See [[Count Set Bits]] for the algorithm.

```cpp
uint32_t value = 0b1010'1100'0011'0101;
int count = std::popcount(value);  // 8

uint64_t big = 0xFFFF'FFFF'0000'0000ULL;
int bigCount = std::popcount(big);  // 32

// constexpr -- can be used at compile time
constexpr int ct = std::popcount(0xFFu);  // 8
```

### std::countl_zero and std::countr_zero -- Leading/Trailing Zeros

```cpp
uint32_t value = 0b0000'0000'0000'0000'0010'0000'0000'0000;  // bit 13
int leading  = std::countl_zero(value);  // 18 leading zeros
int trailing = std::countr_zero(value);  // 13 trailing zeros

// Special cases
int allZeroL = std::countl_zero(uint32_t{0});  // 32
int allZeroT = std::countr_zero(uint32_t{0});  // 32
int allOnesL = std::countl_zero(UINT32_MAX);   // 0
```

### std::countl_one and std::countr_one -- Leading/Trailing Ones

```cpp
uint8_t value = 0b1110'0111;
int leadingOnes  = std::countl_one(value);  // 3 (three consecutive 1s from MSB)
int trailingOnes = std::countr_one(value);  // 3 (three consecutive 1s from LSB)
```

### std::has_single_bit -- Power of Two Check

Returns `true` if the value has exactly one bit set (i.e., is a power of 2). See [[Check Power of Two]].

```cpp
bool yes  = std::has_single_bit(64u);   // true  (0b0100'0000)
bool no   = std::has_single_bit(65u);   // false (0b0100'0001)
bool zero = std::has_single_bit(0u);    // false
```

### std::bit_ceil and std::bit_floor -- Rounding to Power of 2

```cpp
// bit_ceil: round UP to nearest power of 2
uint32_t ceil1 = std::bit_ceil(100u);  // 128
uint32_t ceil2 = std::bit_ceil(128u);  // 128 (already a power of 2)
uint32_t ceil3 = std::bit_ceil(1u);    // 1

// bit_floor: round DOWN to nearest power of 2
uint32_t floor1 = std::bit_floor(100u);  // 64
uint32_t floor2 = std::bit_floor(128u);  // 128 (already a power of 2)
uint32_t floor3 = std::bit_floor(0u);    // 0 (special case)
```

```ad-warning
title: bit_ceil overflow
`std::bit_ceil` can overflow if the result would exceed the type's maximum value. For example, `std::bit_ceil(uint8_t{129})` would need 256, which doesn't fit in `uint8_t`. This is undefined behavior.
```

### std::rotl and std::rotr -- Circular Rotation

Bits that shift out of one end wrap around to the other:

```cpp
uint32_t value = 0b1010'0000'0000'0000'0000'0000'0000'0011;

uint32_t rotL = std::rotl(value, 4);
// 0b0000'0000'0000'0000'0000'0000'0011'1010

uint32_t rotR = std::rotr(value, 4);
// 0b0011'1010'0000'0000'0000'0000'0000'0000

// Negative rotation goes the other direction
uint32_t negRot = std::rotl(value, -4);  // Same as rotr(value, 4)
```

### std::bit_width -- Minimum Bits Needed

Returns the number of bits needed to represent the value (equivalent to `floor(log2(x)) + 1`):

```cpp
int w1 = std::bit_width(0u);    // 0
int w2 = std::bit_width(1u);    // 1
int w3 = std::bit_width(7u);    // 3 (0b111 needs 3 bits)
int w4 = std::bit_width(8u);    // 4 (0b1000 needs 4 bits)
int w5 = std::bit_width(255u);  // 8
int w6 = std::bit_width(256u);  // 9
```

### std::bit_cast -- Type-Safe Reinterpretation

`std::bit_cast` reinterprets the bits of one type as another type, safely and at compile time. It replaces the dangerous `reinterpret_cast` or `memcpy` pattern for type punning.

```cpp
#include <bit>
#include <cstdint>

// See the bit representation of a float
float pi = 3.14f;
uint32_t bits = std::bit_cast<uint32_t>(pi);
// bits = 0x4048F5C3 (IEEE 754 representation)

// Convert back
float restored = std::bit_cast<float>(bits);
// restored = 3.14f

// constexpr -- works at compile time!
constexpr uint64_t double_bits = std::bit_cast<uint64_t>(1.0);
// double_bits = 0x3FF0000000000000
```

```ad-tip
title: bit_cast vs reinterpret_cast
`std::bit_cast` is the only correct and portable way to reinterpret bits between types of the same size. `reinterpret_cast` on values (not pointers) is illegal, and the common `memcpy` workaround cannot be `constexpr`. `std::bit_cast` solves both problems.
```

### std::endian -- Compile-Time Endianness Detection

Detect the system's byte ordering at compile time. See [[Endianness]] for the concept.

```cpp
#include <bit>
#include <iostream>

if constexpr (std::endian::native == std::endian::little) {
    std::cout << "Little-endian system\n";
} else if constexpr (std::endian::native == std::endian::big) {
    std::cout << "Big-endian system\n";
} else {
    std::cout << "Mixed-endian system\n";
}
```

### Complete `<bit>` Header Summary

| Function              | Description                              | Related Note                  |
|-----------------------|------------------------------------------|-------------------------------|
| `std::popcount`       | Count set bits                           | [[Count Set Bits]]            |
| `std::countl_zero`    | Count leading zeros                      |                               |
| `std::countr_zero`    | Count trailing zeros                     |                               |
| `std::countl_one`     | Count leading ones                       |                               |
| `std::countr_one`     | Count trailing ones                      |                               |
| `std::has_single_bit` | Is exactly one bit set (power of 2)?     | [[Check Power of Two]]        |
| `std::bit_ceil`       | Round up to power of 2                   |                               |
| `std::bit_floor`      | Round down to power of 2                 |                               |
| `std::rotl`           | Rotate bits left                         |                               |
| `std::rotr`           | Rotate bits right                        |                               |
| `std::bit_width`      | Minimum bits to represent value          |                               |
| `std::bit_cast`       | Type-safe bit reinterpretation           |                               |
| `std::endian`         | Compile-time endianness detection        | [[Endianness]]                |

---

## Compiler Built-ins (GCC/Clang)

Before C++20's `<bit>` header, GCC and Clang provided compiler intrinsics for common bit operations. These compile to single CPU instructions and are still widely used in C and pre-C++20 code.

### Built-in Functions

```cpp
uint32_t value = 0b1010'1100'0011'0101;

// Count set bits (popcount) -- see [[Count Set Bits]]
int popcount = __builtin_popcount(value);       // uint32_t version
int popcountl = __builtin_popcountl(value);     // unsigned long
int popcountll = __builtin_popcountll(0ULL);    // unsigned long long

// Count leading zeros
int clz = __builtin_clz(value);         // Undefined if value == 0!
int clzl = __builtin_clzl(value);       // unsigned long version
int clzll = __builtin_clzll(0ULL);      // unsigned long long version

// Count trailing zeros
int ctz = __builtin_ctz(value);         // Undefined if value == 0!

// Parity (1 if odd number of set bits, 0 if even)
int parity = __builtin_parity(value);   // 0 (8 set bits = even parity)

// Find first set bit (1-indexed from LSB, 0 if input is 0)
int ffs = __builtin_ffs(0b1010'0000);   // 6 (bit 5 is first set, 1-indexed = 6)
```

```ad-warning
title: __builtin_clz(0) and __builtin_ctz(0) are undefined
Passing 0 to `__builtin_clz` or `__builtin_ctz` is undefined behavior. Always check for zero first, or use the C++20 `std::countl_zero` / `std::countr_zero` which handle zero correctly (returning the bit width).
```

### Built-in vs `<bit>` Comparison

| Built-in               | C++20 `<bit>` Equivalent    | Notes                              |
|------------------------|-----------------------------|------------------------------------|
| `__builtin_popcount`   | `std::popcount`             | Built-in UB on negative signed     |
| `__builtin_clz`        | `std::countl_zero`          | Built-in UB on 0                   |
| `__builtin_ctz`        | `std::countr_zero`          | Built-in UB on 0                   |
| `__builtin_parity`     | (none)                      | Use `std::popcount(x) % 2`        |
| `__builtin_ffs`        | (none)                      | Use `std::countr_zero(x) + 1`     |

```ad-tip
title: Prefer C++20 <bit> when possible
The C++20 `<bit>` functions are type-safe, `constexpr`, handle edge cases (like 0), and are portable across compilers. Use them instead of compiler built-ins whenever C++20 is available.
```

---

## std::bitset

`std::bitset` provides a fixed-size array of bits with a rich API. It is a class template parameterized by the number of bits.

```cpp
#include <bitset>
```

### Creating a bitset

```cpp
// Default: all bits zero
std::bitset<8> bits1;           // 00000000

// From integer
std::bitset<8> bits2(0b1010'0101);  // 10100101

// From string
std::bitset<8> bits3("11001100");   // 11001100

// From string with custom characters
std::bitset<8> bits4(std::string("XOXOXXOO"), 0, 8, 'O', 'X');
// O=0, X=1: 10101100
```

### Accessing Individual Bits

```cpp
std::bitset<8> bits("10110010");

// Test a bit (read-only, bounds-checked)
bool bit0 = bits.test(0);   // false (bit 0 = '0')
bool bit1 = bits.test(1);   // true  (bit 1 = '1')

// Operator[] (no bounds checking, slightly faster)
bool bit7 = bits[7];        // true  (MSB)

// Set a bit to 1
bits.set(0);       // Set bit 0 to 1
bits.set(3, true); // Set bit 3 to 1

// Reset a bit to 0
bits.reset(1);     // Set bit 1 to 0

// Flip a bit (toggle)
bits.flip(4);      // Toggle bit 4
```

### Bulk Operations

```cpp
std::bitset<8> bits("10110010");

bits.set();     // Set ALL bits to 1: 11111111
bits.reset();   // Set ALL bits to 0: 00000000
bits.flip();    // Flip ALL bits:     11111111 (was all 0)
```

### Querying

```cpp
std::bitset<8> bits("10110010");

int count = bits.count();   // 4 (number of 1-bits) -- see [[Count Set Bits]]
int size  = bits.size();    // 8 (total number of bits)

bool allSet  = bits.all();  // false (not all bits are 1)
bool anySet  = bits.any();  // true  (at least one bit is 1)
bool noneSet = bits.none(); // false (not all bits are 0)
```

### Bitwise Operations on bitsets

The full set of bitwise operators work on bitsets of the same size:

```cpp
std::bitset<8> a("10101010");
std::bitset<8> b("11001100");

std::bitset<8> andResult = a & b;  // 10001000 -- see [[AND Operator]]
std::bitset<8> orResult  = a | b;  // 11101110 -- see [[OR Operator]]
std::bitset<8> xorResult = a ^ b;  // 01100110 -- see [[XOR Operator]]
std::bitset<8> notResult = ~a;     // 01010101 -- see [[NOT Operator]]

// Shift operations
std::bitset<8> leftShift  = a << 2;  // 10101000 -- see [[Left Shift]]
std::bitset<8> rightShift = a >> 2;  // 00101010 -- see [[Right Shift]]
```

### Conversions

```cpp
std::bitset<8> bits("10110010");

// To string
std::string str = bits.to_string();        // "10110010"
std::string custom = bits.to_string('O', 'X');  // "XOXXOOXO"

// To integer (throws if value exceeds type range)
unsigned long  val  = bits.to_ulong();     // 178
unsigned long long val2 = bits.to_ullong(); // 178
```

```ad-note
title: bitset size is fixed at compile time
Unlike C#'s `BitArray` which has a dynamic size, `std::bitset` requires the size as a template parameter, so it must be known at compile time. For a dynamic-size bit array, consider `std::vector<bool>` (though it has its own quirks) or a third-party library.
```

---

## Bit Fields in Structs

C++ allows declaring struct members with specific bit widths, packing multiple values into a single word.

### Basic Usage

```cpp
struct FilePermissions {
    unsigned int read    : 1;  // 1 bit
    unsigned int write   : 1;  // 1 bit
    unsigned int execute : 1;  // 1 bit
};

FilePermissions perms;
perms.read = 1;
perms.write = 0;
perms.execute = 1;

// sizeof(FilePermissions) is typically 4 bytes (compiler pads to word boundary)
```

### Packing Multiple Fields

```cpp
struct Color {
    uint32_t red   : 8;   // 0-255
    uint32_t green : 8;   // 0-255
    uint32_t blue  : 8;   // 0-255
    uint32_t alpha : 8;   // 0-255
};
// Total: 32 bits = 4 bytes (fits in one uint32_t)

Color c;
c.red = 255;
c.green = 128;
c.blue = 0;
c.alpha = 255;

// Access like normal struct members
std::cout << "Red: " << c.red << std::endl;
```

### Unnamed Bit Fields (Padding)

```cpp
struct Packed {
    uint16_t field1 : 3;   // 3 bits
    uint16_t        : 5;   // 5 bits of padding (unnamed field)
    uint16_t field2 : 8;   // 8 bits
};
```

```ad-warning
title: Portability concerns with bit fields
Bit fields are one of the least portable features in C++. The standard does NOT specify:
- The order in which bits are allocated within a byte (MSB-first or LSB-first)
- Whether bit fields can straddle storage unit boundaries
- The exact padding between bit fields

Different compilers on different platforms may lay out the same bit field struct differently. Never use bit fields for data exchanged across systems (binary file formats, network protocols). Use explicit bitwise operations with [[Creating Bit Masks]] instead.
```

```ad-tip
title: When to use bit fields
Bit fields are useful for:
- Internal hardware register definitions (for a specific compiler/platform)
- Reducing memory usage in large arrays of small structs
- Matching vendor-specific binary layouts (with compiler-specific attributes)

For portable code, prefer explicit bitmasks with `&`, `|`, `^`, and shifts.
```

---

## Byte Swapping and Endianness Conversion

For converting between byte orders, use platform-specific functions or C++23's `std::byteswap`. See [[Endianness]] for the concept.

### C++23: std::byteswap

```cpp
#include <bit>  // C++23

uint32_t value = 0x01020304;
uint32_t swapped = std::byteswap(value);  // 0x04030201
```

### Pre-C++23: Compiler-Specific

```cpp
// GCC/Clang
uint32_t swapped32 = __builtin_bswap32(0x01020304);  // 0x04030201
uint64_t swapped64 = __builtin_bswap64(0x0102030405060708ULL);

// MSVC
#include <cstdlib>
uint32_t swapped = _byteswap_ulong(0x01020304);
uint64_t swapped64 = _byteswap_uint64(0x0102030405060708ULL);
```

### Manual Byte Swap

```cpp
constexpr uint32_t byte_swap(uint32_t value) {
    return ((value & 0xFF000000u) >> 24) |
           ((value & 0x00FF0000u) >>  8) |
           ((value & 0x0000FF00u) <<  8) |
           ((value & 0x000000FFu) << 24);
}
```

---

## Printing Binary Representations

### Using std::bitset for Display

```cpp
#include <bitset>
#include <iostream>

uint32_t value = 0xDEAD;

// Full 32-bit binary
std::cout << std::bitset<32>(value) << std::endl;
// 00000000000000001101111010101101

// 16-bit view
std::cout << std::bitset<16>(value) << std::endl;
// 1101111010101101

// Hex output
std::cout << std::hex << value << std::endl;  // dead
std::cout << "0x" << std::uppercase << std::hex << value << std::endl;  // 0xDEAD
```

### Grouped Binary Display Helper

```cpp
#include <string>
#include <sstream>

template<typename T>
std::string to_binary_grouped(T value, int group_size = 4) {
    constexpr int bits = sizeof(T) * 8;
    std::string binary = std::bitset<64>(value).to_string().substr(64 - bits);
    std::string result;
    for (int i = 0; i < bits; i++) {
        if (i > 0 && i % group_size == 0) result += '\'';
        result += binary[i];
    }
    return result;
}

// Usage:
std::cout << to_binary_grouped(uint8_t{0xA5}) << std::endl;
// 1010'0101

std::cout << to_binary_grouped(uint16_t{0xDEAD}) << std::endl;
// 1101'1110'1010'1101
```

---

## Complete Example: Combining C++ Bit Features

```cpp
#include <bit>
#include <bitset>
#include <cstdint>
#include <iostream>

// Using bit fields for internal representation
struct PackedFlags {
    uint32_t readable   : 1;
    uint32_t writable   : 1;
    uint32_t executable : 1;
    uint32_t hidden     : 1;
    uint32_t system     : 1;
    uint32_t reserved   : 27;
};

// Using explicit bitmasks for portable serialization
namespace PortableFlags {
    constexpr uint32_t READABLE   = 1u << 0;  // See [[Creating Bit Masks]]
    constexpr uint32_t WRITABLE   = 1u << 1;
    constexpr uint32_t EXECUTABLE = 1u << 2;
    constexpr uint32_t HIDDEN     = 1u << 3;
    constexpr uint32_t SYSTEM     = 1u << 4;

    // Check, set, clear -- see [[Check Set and Clear Bits]]
    constexpr bool has_flag(uint32_t flags, uint32_t flag) {
        return (flags & flag) != 0;
    }
    constexpr uint32_t set_flag(uint32_t flags, uint32_t flag) {
        return flags | flag;
    }
    constexpr uint32_t clear_flag(uint32_t flags, uint32_t flag) {
        return flags & ~flag;
    }
    constexpr uint32_t toggle_flag(uint32_t flags, uint32_t flag) {
        return flags ^ flag;  // See [[Toggle Bits]]
    }
}

int main() {
    using namespace PortableFlags;

    uint32_t flags = READABLE | WRITABLE | HIDDEN;
    std::cout << "Flags: " << std::bitset<8>(flags) << std::endl;
    // Flags: 00001011

    std::cout << "Readable? " << has_flag(flags, READABLE) << std::endl;  // 1
    std::cout << "Executable? " << has_flag(flags, EXECUTABLE) << std::endl;  // 0

    flags = set_flag(flags, EXECUTABLE);
    flags = clear_flag(flags, HIDDEN);
    std::cout << "After modify: " << std::bitset<8>(flags) << std::endl;
    // After modify: 00000111

    // C++20 <bit> usage
    std::cout << "Active flags: " << std::popcount(flags) << std::endl;  // 3
    std::cout << "Lowest flag bit: " << std::countr_zero(flags) << std::endl;  // 0
    std::cout << "Is power of 2: " << std::has_single_bit(flags) << std::endl;  // 0

    // Endianness check
    if constexpr (std::endian::native == std::endian::little) {
        std::cout << "Running on little-endian system" << std::endl;
    }

    return 0;
}
```

---

## Quick Reference Summary

| Feature                    | Header / Requirement      | Standard Version          |
|----------------------------|---------------------------|---------------------------|
| Fixed-width types          | `<cstdint>`               | C++11                     |
| Binary literals (`0b`)    | Language feature           | C++14                     |
| Digit separators (`'`)    | Language feature           | C++14                     |
| `std::bitset`             | `<bitset>`                 | C++98                     |
| Bit fields                | Language feature           | C++98                     |
| `__builtin_popcount` etc. | GCC/Clang built-in         | N/A (non-standard)        |
| `<bit>` header            | `<bit>`                    | C++20                     |
| `std::bit_cast`           | `<bit>`                    | C++20                     |
| `std::endian`             | `<bit>`                    | C++20                     |
| Two's complement defined  | Language semantics          | C++20                     |
| `std::byteswap`           | `<bit>`                    | C++23                     |

See also: [[Bit Manipulation in CSharp]] for the C# perspective (guaranteed type sizes, no UB concerns), and [[Bit Manipulation in JavaScript]] for JavaScript's unique 32-bit conversion behavior.
