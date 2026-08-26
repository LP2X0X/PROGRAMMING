---
tags:
  - bit-manipulation
  - binary
  - number-system
---

## What Is Binary?

Binary is a **base-2** numeral system that uses only two digits: `0` and `1`. Each digit in a binary number is called a **bit** (binary digit). Every piece of data a computer processes — text, images, audio, instructions — is ultimately represented as sequences of bits.

---

## Why Computers Use Binary

At the hardware level, computers are built from **transistors** — tiny electronic switches that are either **on** or **off**. These two states map naturally to two voltage levels:

| State | Voltage (typical TTL) | Binary Digit |
|-------|----------------------|--------------|
| Off   | 0V (ground)          | `0`          |
| On    | ~3.3V or ~5V         | `1`          |

```ad-tip
title: Why not base-10?
Building circuits that reliably distinguish between 10 different voltage levels is far more error-prone and expensive than distinguishing between just 2. Binary's simplicity makes hardware cheaper, faster, and more reliable.
```

---

## Positional Notation

Binary uses **positional notation**, just like decimal. Each position represents a power of 2 (instead of a power of 10).

**Decimal (base-10) refresher:**

```
  4    7    3
  |    |    |
  |    |    +-- 3 x 10^0 = 3 x 1   =   3
  |    +------- 7 x 10^1 = 7 x 10  =  70
  +------------ 4 x 10^2 = 4 x 100 = 400
                                      ---
                                      473
```

**Binary (base-2):**

```
  1    0    1    1    0    1
  |    |    |    |    |    |
  |    |    |    |    |    +-- 1 x 2^0 = 1 x 1  =  1
  |    |    |    |    +------- 0 x 2^1 = 0 x 2  =  0
  |    |    |    +------------ 1 x 2^2 = 1 x 4  =  4
  |    |    +----------------- 1 x 2^3 = 1 x 8  =  8
  |    +---------------------- 0 x 2^4 = 0 x 16 =  0
  +--------------------------- 1 x 2^5 = 1 x 32 = 32
                                                  --
                                                  45
```

So `101101` in binary equals `45` in decimal.

---

## Powers of 2 Reference Table

Memorizing the first several powers of 2 is essential for working with bits:

| Power | Value   | Common Name |
|-------|---------|-------------|
| 2^0   | 1       |             |
| 2^1   | 2       |             |
| 2^2   | 4       |             |
| 2^3   | 8       | Nibble max+1|
| 2^4   | 16      |             |
| 2^5   | 32      |             |
| 2^6   | 64      |             |
| 2^7   | 128     |             |
| 2^8   | 256     | Byte max+1  |
| 2^10  | 1,024   | 1 KiB       |
| 2^16  | 65,536  |             |
| 2^20  | 1,048,576| 1 MiB      |
| 2^32  | 4,294,967,296 | ~4 GiB|

---

## Converting Decimal to Binary

### Method 1: Repeated Division by 2

Divide the decimal number by 2 repeatedly, recording the remainder at each step. Read the remainders from bottom to top.

**Example: Convert 45 to binary**

```
45 / 2 = 22  remainder 1   <-- LSB (least significant bit)
22 / 2 = 11  remainder 0
11 / 2 = 5   remainder 1
 5 / 2 = 2   remainder 1
 2 / 2 = 1   remainder 0
 1 / 2 = 0   remainder 1   <-- MSB (most significant bit)

Read bottom to top: 101101
```

### Method 2: Subtract Largest Powers of 2

Find the largest power of 2 that fits, subtract it, repeat.

**Example: Convert 200 to binary**

```
200 - 128 (2^7) = 72   --> bit 7 = 1
 72 -  64 (2^6) =  8   --> bit 6 = 1
  8 -   8 (2^3) =  0   --> bit 3 = 1
All other bits = 0

Result: 11001000
```

---

## Converting Binary to Decimal

Multiply each bit by its positional power of 2, then sum:

**Example: Convert `11001000` to decimal**

```
1x128 + 1x64 + 0x32 + 0x16 + 1x8 + 0x4 + 0x2 + 0x1
= 128 + 64 + 0 + 0 + 8 + 0 + 0 + 0
= 200
```

---

## Binary Addition

Binary addition follows the same carry rules as decimal, but simpler:

| A | B | Sum | Carry |
|---|---|-----|-------|
| 0 | 0 |  0  |   0   |
| 0 | 1 |  1  |   0   |
| 1 | 0 |  1  |   0   |
| 1 | 1 |  0  |   1   |

**Example: 1011 + 0110**

```
    1 1        <-- carries
    1 0 1 1    (11 in decimal)
  + 0 1 1 0    ( 6 in decimal)
  ---------
  1 0 0 0 1    (17 in decimal)
```

```ad-warning
title: Overflow
If you are working with a fixed number of bits (e.g., 8-bit), the carry out of the most significant bit causes **overflow**. The result wraps around. See [[Signed and Unsigned Integers]] for more on overflow behavior.
```

---

## Binary Literals and Conversions in Code

### C++

```cpp
#include <iostream>
#include <bitset>
#include <cstdint>

int main() {
    // Binary literal (C++14 and later)
    int value = 0b101101;  // 45

    // Digit separator for readability (C++14)
    int large = 0b1100'1000;  // 200

    // Print decimal value
    std::cout << "Decimal: " << value << std::endl;  // 45

    // Print binary representation using bitset
    std::cout << "Binary:  " << std::bitset<8>(value) << std::endl;  // 00101101

    // Convert string to integer (binary string)
    int fromStr = std::stoi("101101", nullptr, 2);
    std::cout << "From string: " << fromStr << std::endl;  // 45

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
        // Binary literal (C# 7.0+)
        int value = 0b101101;  // 45

        // Digit separator for readability (C# 7.0+)
        int large = 0b1100_1000;  // 200

        // Print decimal value
        Console.WriteLine($"Decimal: {value}");  // 45

        // Print binary representation
        Console.WriteLine($"Binary:  {Convert.ToString(value, 2)}");          // 101101
        Console.WriteLine($"Binary:  {Convert.ToString(value, 2).PadLeft(8, '0')}");  // 00101101

        // Convert binary string to integer
        int fromStr = Convert.ToInt32("101101", 2);
        Console.WriteLine($"From string: {fromStr}");  // 45
    }
}
```

### JavaScript

```javascript
// Binary literal (ES2015+)
let value = 0b101101;  // 45

// Print decimal value
console.log(`Decimal: ${value}`);  // 45

// Print binary representation
console.log(`Binary:  ${value.toString(2)}`);                // 101101
console.log(`Binary:  ${value.toString(2).padStart(8, '0')}`);  // 00101101

// Convert binary string to integer
let fromStr = parseInt("101101", 2);
console.log(`From string: ${fromStr}`);  // 45

// Note: JavaScript numbers are IEEE 754 doubles,
// but bitwise operators treat them as 32-bit signed integers.
// See [[Signed and Unsigned Integers]] for implications.
```

---

## Common Binary Patterns Worth Recognizing

| Pattern          | Decimal | Significance                           |
|------------------|---------|----------------------------------------|
| `0000 0000`      | 0       | All bits clear                         |
| `1111 1111`      | 255     | All bits set (8-bit unsigned max)      |
| `1000 0000`      | 128     | Only MSB set                           |
| `0000 0001`      | 1       | Only LSB set                           |
| `0111 1111`      | 127     | All bits set except MSB (signed max)   |
| `1010 1010`      | 170     | Alternating bits (test pattern)        |
| `0101 0101`      | 85      | Alternating bits (complement)          |

```ad-tip
title: Quick mental trick
A binary number that is all 1s for N bits equals `2^N - 1`. For example, `1111` (4 bits) = `2^4 - 1 = 15`, and `11111111` (8 bits) = `2^8 - 1 = 255`.
```

---

## Relationship to Other Number Systems

Binary digits group neatly into larger bases:
- **4 bits** = 1 [[Hexadecimal and Octal|hexadecimal]] digit (base-16)
- **3 bits** = 1 [[Hexadecimal and Octal|octal]] digit (base-8)

This grouping makes hex and octal convenient shorthand for writing binary values. See [[Hexadecimal and Octal]] for details.

For how bits combine into larger data units, see [[Bits Bytes and Words]].
