---
tags:
  - bit-manipulation
  - fundamental
  - terminology
---

## The Data Size Hierarchy

Every piece of data in a computer is built from **bits**. Bits group into standard-sized units that correspond to hardware capabilities and data type conventions.

```
Bit  ->  Nibble  ->  Byte  ->  Word  ->  Double Word  ->  Quad Word
 1        4          8         16/32/64     32/64          64/128
```

---

## Bit

A **bit** (binary digit) is the smallest unit of data. It holds exactly one of two values: `0` or `1`.

- A single bit can represent: true/false, on/off, yes/no
- A bit has **2^1 = 2** possible states

---

## Nibble

A **nibble** (also spelled "nybble") is **4 bits**.

- A nibble can represent **2^4 = 16** values (0 through 15)
- A nibble maps to exactly **one hexadecimal digit** (see [[Hexadecimal and Octal]])
- Two nibbles = one byte

```
Nibble:   1010
Hex:      0xA
Decimal:  10
```

```ad-tip
title: Where you see nibbles
Nibbles appear in BCD (Binary-Coded Decimal), where each decimal digit is encoded as a 4-bit nibble. They are also visible whenever you read hex dumps — each hex character is one nibble.
```

---

## Byte

A **byte** is **8 bits** (2 nibbles). It is the fundamental addressable unit of memory in virtually all modern computers.

- A byte can represent **2^8 = 256** values (0 through 255 unsigned)
- One byte = exactly **2 hexadecimal digits** (`0x00` to `0xFF`)
- One ASCII character fits in one byte

```
Byte:     1010 0011
           |     |
         nibble nibble
Hex:      0xA3
Decimal:  163
```

### The Byte as a Unit of Measurement

| Unit     | Bytes (decimal, SI) | Bytes (binary, IEC)     |
|----------|--------------------|-----------------------|
| 1 KB     | 1,000              | -                     |
| 1 KiB    | -                  | 1,024 (2^10)          |
| 1 MB     | 1,000,000           | -                     |
| 1 MiB    | -                  | 1,048,576 (2^20)      |
| 1 GB     | 1,000,000,000       | -                     |
| 1 GiB    | -                  | 1,073,741,824 (2^30)  |
| 1 TB     | 1,000,000,000,000   | -                     |
| 1 TiB    | -                  | 1,099,511,627,776 (2^40) |

```ad-warning
title: KB vs KiB
Disk manufacturers use SI (powers of 10), while operating systems and programmers typically use binary (powers of 2). A "500 GB" drive has 500,000,000,000 bytes, which is about 465 GiB. This discrepancy confuses users and is worth being precise about in documentation.
```

---

## Word

A **word** is the natural data unit for a given CPU architecture. The word size determines how much data the CPU can process in a single operation.

| Architecture   | Word Size | Typical Register Width |
|----------------|-----------|----------------------|
| 8-bit (AVR)    | 8 bits    | 8-bit registers      |
| 16-bit (8086)  | 16 bits   | 16-bit registers     |
| 32-bit (x86)   | 32 bits   | 32-bit registers     |
| 64-bit (x64)   | 64 bits   | 64-bit registers     |

```ad-note
title: x86 naming convention
Intel's x86 documentation uses specific names that originated from the 16-bit era:
- **WORD** = 16 bits (even on 32/64-bit systems)
- **DWORD** (Double Word) = 32 bits
- **QWORD** (Quad Word) = 64 bits

This naming is historical. When Intel moved from 16 to 32 bits, they kept "WORD" at 16 bits for backward compatibility. Don't confuse Intel's "WORD" with the general concept of a CPU's native word size.
```

---

## Double Word and Quad Word

| Name         | Size     | Bits | Hex Digits | Common Use              |
|--------------|----------|------|------------|-------------------------|
| Byte         | 1 byte   | 8    | 2          | Characters, small flags  |
| Word (Intel) | 2 bytes  | 16   | 4          | Legacy 16-bit values     |
| DWORD        | 4 bytes  | 32   | 8          | 32-bit integers, floats  |
| QWORD        | 8 bytes  | 64   | 16         | 64-bit integers, pointers|

---

## Word Size and CPU Architecture

The word size of a CPU affects:

1. **Register width**: How many bits each general-purpose register can hold
2. **Memory addressing**: How much RAM the system can directly address
3. **Data bus width**: How many bits transfer per memory access
4. **Instruction pointer size**: How large the program counter is

### Memory Addressing Limits

```
32-bit CPU: 2^32 addresses = 4,294,967,296 bytes = 4 GiB max RAM
64-bit CPU: 2^64 addresses = 18,446,744,073,709,551,616 bytes = 16 EiB (theoretical)
            (practical limit: 2^48 = 256 TiB on current x64 implementations)
```

```
32-bit address space:

Address:   00000000  to  FFFFFFFF
           |                    |
           0 bytes         4 GiB - 1

64-bit address space (48-bit practical):

Address:   0000000000000000  to  0000FFFFFFFFFFFF
           |                                    |
           0 bytes                        256 TiB - 1
```

---

## Data Type Sizes Across Languages

### C++

C++ data type sizes are **implementation-defined** with guaranteed minimums:

| Type                | Minimum Size | Typical (x64) | Range (typical)                   |
|---------------------|-------------|----------------|-----------------------------------|
| `char`              | 8 bits      | 8 bits         | -128 to 127 or 0 to 255          |
| `short`             | 16 bits     | 16 bits        | -32,768 to 32,767                 |
| `int`               | 16 bits     | 32 bits        | -2,147,483,648 to 2,147,483,647   |
| `long`              | 32 bits     | 32 or 64 bits  | Platform-dependent                |
| `long long`         | 64 bits     | 64 bits        | -9.2 x 10^18 to 9.2 x 10^18     |
| `float`             | -           | 32 bits        | ~7 decimal digits precision       |
| `double`            | -           | 64 bits        | ~15 decimal digits precision      |
| pointer             | -           | 64 bits (x64)  | Depends on architecture           |

```cpp
#include <iostream>
#include <cstdint>  // For fixed-width types
#include <climits>

int main() {
    // Check sizes at runtime
    std::cout << "char:      " << sizeof(char)      << " bytes" << std::endl;  // 1
    std::cout << "short:     " << sizeof(short)     << " bytes" << std::endl;  // 2
    std::cout << "int:       " << sizeof(int)       << " bytes" << std::endl;  // 4
    std::cout << "long:      " << sizeof(long)      << " bytes" << std::endl;  // 4 or 8
    std::cout << "long long: " << sizeof(long long) << " bytes" << std::endl;  // 8
    std::cout << "pointer:   " << sizeof(void*)     << " bytes" << std::endl;  // 4 or 8

    // Fixed-width integer types (guaranteed sizes)
    int8_t   a = 42;    // Exactly 8 bits
    int16_t  b = 42;    // Exactly 16 bits
    int32_t  c = 42;    // Exactly 32 bits
    int64_t  d = 42;    // Exactly 64 bits
    uint32_t e = 42;    // Exactly 32 bits, unsigned

    std::cout << "int8_t:  " << sizeof(a) << " byte(s)" << std::endl;   // 1
    std::cout << "int64_t: " << sizeof(d) << " byte(s)" << std::endl;   // 8

    return 0;
}
```

```ad-warning
title: C++ size guarantees
In C++, `sizeof(char) == 1` is the ONLY hard guarantee on exact size. Everything else is a minimum. Use `<cstdint>` types like `int32_t` when you need exact sizes. See [[Signed and Unsigned Integers]] for signed vs unsigned variants.
```

### C#

C# has **fixed** data type sizes regardless of platform:

| Type      | Size     | Bits | Range                                     |
|-----------|----------|------|-------------------------------------------|
| `byte`    | 1 byte   | 8    | 0 to 255                                  |
| `sbyte`   | 1 byte   | 8    | -128 to 127                                |
| `short`   | 2 bytes  | 16   | -32,768 to 32,767                          |
| `ushort`  | 2 bytes  | 16   | 0 to 65,535                                |
| `int`     | 4 bytes  | 32   | -2,147,483,648 to 2,147,483,647            |
| `uint`    | 4 bytes  | 32   | 0 to 4,294,967,295                         |
| `long`    | 8 bytes  | 64   | -9.2 x 10^18 to 9.2 x 10^18              |
| `ulong`   | 8 bytes  | 64   | 0 to 18.4 x 10^18                         |
| `float`   | 4 bytes  | 32   | ~6-9 digits precision                      |
| `double`  | 8 bytes  | 64   | ~15-17 digits precision                    |
| `nint`    | 4 or 8   | 32/64| Platform-dependent (native int)            |

```csharp
using System;
using System.Runtime.InteropServices;

class Program
{
    static void Main()
    {
        // All sizes are fixed in C#
        Console.WriteLine($"byte:    {sizeof(byte)} byte(s)");     // 1
        Console.WriteLine($"short:   {sizeof(short)} byte(s)");    // 2
        Console.WriteLine($"int:     {sizeof(int)} byte(s)");      // 4
        Console.WriteLine($"long:    {sizeof(long)} byte(s)");     // 8
        Console.WriteLine($"float:   {sizeof(float)} byte(s)");    // 4
        Console.WriteLine($"double:  {sizeof(double)} byte(s)");   // 8

        // Pointer size depends on platform
        Console.WriteLine($"IntPtr:  {IntPtr.Size} byte(s)");      // 4 (32-bit) or 8 (64-bit)

        // Min/Max values
        Console.WriteLine($"int range: {int.MinValue} to {int.MaxValue}");
        Console.WriteLine($"byte range: {byte.MinValue} to {byte.MaxValue}");
    }
}
```

### JavaScript

JavaScript has a **single number type**: IEEE 754 double-precision floating-point (64 bits). However, bitwise operations internally use **32-bit signed integers**.

| Concept            | Size     | Bits | Notes                                      |
|--------------------|----------|------|--------------------------------------------|
| `number`           | 8 bytes  | 64   | IEEE 754 double (all numbers)              |
| Bitwise operand    | 4 bytes  | 32   | Internally cast for `&`, `|`, `^`, `~`, shifts |
| `BigInt`           | Variable | N/A  | Arbitrary precision (ES2020+)              |
| `ArrayBuffer` byte | 1 byte   | 8    | Raw binary data                            |

```javascript
// JavaScript's number is always a 64-bit double
console.log(typeof 42);        // "number"
console.log(typeof 3.14);      // "number"

// Typed arrays give you fixed-size integer views
let buffer = new ArrayBuffer(8);  // 8 bytes of raw memory

let uint8  = new Uint8Array(buffer);     // View as 8 x 1-byte unsigned
let int32  = new Int32Array(buffer);     // View as 2 x 4-byte signed
let uint16 = new Uint16Array(buffer);    // View as 4 x 2-byte unsigned

console.log(uint8.BYTES_PER_ELEMENT);    // 1
console.log(int32.BYTES_PER_ELEMENT);    // 4
console.log(uint16.BYTES_PER_ELEMENT);   // 2

// Safe integer range for exact representation
console.log(Number.MAX_SAFE_INTEGER);    // 9007199254740991 (2^53 - 1)
console.log(Number.MIN_SAFE_INTEGER);    // -9007199254740991

// BigInt for arbitrary precision (ES2020+)
let big = 9007199254740992n;  // Beyond MAX_SAFE_INTEGER
console.log(big + 1n);       // 9007199254740993n
```

```ad-warning
title: JavaScript bitwise operations are 32-bit
Even though JavaScript numbers are 64-bit doubles, ALL bitwise operators (`&`, `|`, `^`, `~`, `<<`, `>>`, `>>>`) convert their operands to **32-bit signed integers** first, then convert back. This means you lose precision on numbers larger than 2^31 - 1 when using bitwise operations. See [[Signed and Unsigned Integers]] for details.
```

---

## Memory Layout Visualization

Here is how data types occupy memory, shown byte-by-byte:

```
Address:  0x00  0x01  0x02  0x03  0x04  0x05  0x06  0x07
         +-----+-----+-----+-----+-----+-----+-----+-----+
byte:    | 0xA3|     |     |     |     |     |     |     |
         +-----+-----+-----+-----+-----+-----+-----+-----+
short:   | 0xA3| 0x4F|     |     |     |     |     |     |
         +-----+-----+-----+-----+-----+-----+-----+-----+
int:     | 0xA3| 0x4F| 0x12| 0x00|     |     |     |     |
         +-----+-----+-----+-----+-----+-----+-----+-----+
long:    | 0xA3| 0x4F| 0x12| 0x00| 0x78| 0x56| 0x34| 0x12|
         +-----+-----+-----+-----+-----+-----+-----+-----+
```

```ad-note
title: Byte order matters
The order in which bytes are stored for multi-byte types depends on the system's [[Endianness]]. The diagram above shows one possible arrangement — real layouts differ between big-endian and little-endian systems.
```

---

## Alignment and Padding

CPUs access memory most efficiently when data is aligned to its natural size boundary:

| Data Size  | Natural Alignment |
|-----------|------------------|
| 1 byte    | Any address       |
| 2 bytes   | Even address (divisible by 2) |
| 4 bytes   | Address divisible by 4 |
| 8 bytes   | Address divisible by 8 |

Compilers insert **padding bytes** between struct/class members to maintain alignment:

```
struct Example {
    char  a;     // 1 byte  at offset 0
                 // 3 bytes padding
    int   b;     // 4 bytes at offset 4  (aligned to 4)
    char  c;     // 1 byte  at offset 8
                 // 7 bytes padding
    double d;    // 8 bytes at offset 16 (aligned to 8)
};
// Total: 24 bytes (not 14!)
```

```ad-tip
title: Minimize padding
Order struct members from largest to smallest to minimize wasted padding bytes. In the example above, reordering as `double d; int b; char a; char c;` would reduce the struct to 16 bytes.
```

---

## Related Notes

- [[Binary Number System]] — the base-2 foundation
- [[Signed and Unsigned Integers]] — how interpretation changes with signed/unsigned
- [[Endianness]] — byte ordering in multi-byte values
- [[Hexadecimal and Octal]] — shorthand notations for binary
