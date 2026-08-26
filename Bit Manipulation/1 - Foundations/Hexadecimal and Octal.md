---
tags:
  - bit-manipulation
  - hexadecimal
  - octal
  - number-system
---

## Why Other Bases?

Writing out long strings of `0`s and `1`s is tedious and error-prone. **Hexadecimal** (base-16) and **octal** (base-8) serve as compact shorthands for binary because they group bits evenly:

- **1 hex digit** = exactly **4 bits** (a nibble)
- **1 octal digit** = exactly **3 bits**

This makes conversion between binary and hex/octal trivial — you just group and translate.

---

## Hexadecimal (Base-16)

Hex uses 16 symbols: `0-9` and `A-F` (or `a-f`).

### Hex-to-Binary Mapping

| Hex | Binary | Decimal |   | Hex | Binary | Decimal |
|-----|--------|---------|---|-----|--------|---------|
| 0   | 0000   | 0       |   | 8   | 1000   | 8       |
| 1   | 0001   | 1       |   | 9   | 1001   | 9       |
| 2   | 0010   | 2       |   | A   | 1010   | 10      |
| 3   | 0011   | 3       |   | B   | 1011   | 11      |
| 4   | 0100   | 4       |   | C   | 1100   | 12      |
| 5   | 0101   | 5       |   | D   | 1101   | 13      |
| 6   | 0110   | 6       |   | E   | 1110   | 14      |
| 7   | 0111   | 7       |   | F   | 1111   | 15      |

### Converting Binary to Hex

Group the binary digits into nibbles (4-bit groups) from right to left, then replace each group:

```
Binary:  1100 1010 1111 0010
Hex:        C    A    F    2

Result: 0xCAF2
```

```ad-tip
title: Pad with leading zeros
If the leftmost group has fewer than 4 bits, pad with leading zeros. For example: `10111` becomes `0001 0111` = `0x17`.
```

### Converting Hex to Binary

Replace each hex digit with its 4-bit binary equivalent:

```
Hex:     0x3F
Binary:  0011 1111

Hex:     0xFF
Binary:  1111 1111
```

### Converting Hex to Decimal

Multiply each hex digit by its positional power of 16:

```
0x2A = 2 x 16^1 + A(10) x 16^0
     = 32 + 10
     = 42
```

### Why Hex Is So Common

- An **8-bit byte** is exactly **2 hex digits**: `0x00` to `0xFF`
- A **32-bit value** is exactly **8 hex digits**: `0x00000000` to `0xFFFFFFFF`
- Memory addresses, color codes (`#FF8800`), MAC addresses — all use hex
- Much easier to spot bit patterns than in decimal

```ad-note
title: Color example
The CSS color `#1A2B3C` breaks down as:
- Red:   `0x1A` = `0001 1010` = 26
- Green: `0x2B` = `0010 1011` = 43
- Blue:  `0x3C` = `0011 1100` = 60

See [[Color and Graphics]] for more on how bits encode colors.
```

---

## Octal (Base-8)

Octal uses 8 symbols: `0-7`.

### Octal-to-Binary Mapping

| Octal | Binary | Decimal |
|-------|--------|---------|
| 0     | 000    | 0       |
| 1     | 001    | 1       |
| 2     | 010    | 2       |
| 3     | 011    | 3       |
| 4     | 100    | 4       |
| 5     | 101    | 5       |
| 6     | 110    | 6       |
| 7     | 111    | 7       |

### Converting Binary to Octal

Group binary digits into 3-bit groups from right to left:

```
Binary:  1  100  101  011
Octal:   1    4    5    3

Result: 01453
```

### Converting Octal to Decimal

```
0o755 = 7 x 8^2 + 5 x 8^1 + 5 x 8^0
      = 448 + 40 + 5
      = 493
```

### Where Octal Is Used

Octal is less common than hex in modern programming, but it still appears in:

- **Unix file permissions**: `chmod 755` means `rwxr-xr-x`
  ```
  7 = 111 = rwx  (owner: read + write + execute)
  5 = 101 = r-x  (group: read + execute)
  5 = 101 = r-x  (other: read + execute)
  ```
- **Legacy C code**: octal literals start with `0` (e.g., `0755`)

```ad-warning
title: Accidental octal in C/C++
A leading `0` in C/C++ makes a number octal, not decimal. Writing `int x = 010;` sets `x` to **8**, not **10**. This is a common source of bugs.
```

---

## Prefix Notation Across Languages

| Base        | C++            | C#             | JavaScript      |
|-------------|----------------|----------------|-----------------|
| Binary      | `0b` or `0B`   | `0b` or `0B`   | `0b` or `0B`    |
| Octal       | `0` (legacy) or `0` | N/A (no octal literal) | `0o` or `0O` |
| Decimal     | (no prefix)    | (no prefix)    | (no prefix)     |
| Hexadecimal | `0x` or `0X`   | `0x` or `0X`   | `0x` or `0X`    |

```ad-note
title: C# and Octal
C# intentionally has **no octal literal** syntax. You must use `Convert.ToInt32("755", 8)` to parse octal strings, or write hex/binary instead.
```

---

## Code Examples

### C++

```cpp
#include <iostream>
#include <bitset>
#include <iomanip>

int main() {
    // Hex literal
    int hex_val = 0xFF;       // 255
    // Binary literal (C++14)
    int bin_val = 0b11111111; // 255
    // Octal literal (legacy prefix: leading 0)
    int oct_val = 0377;       // 255

    std::cout << "All three represent: " << hex_val << std::endl;

    // Print in different bases
    std::cout << "Hex: " << std::hex << hex_val << std::endl;         // ff
    std::cout << "Oct: " << std::oct << hex_val << std::endl;         // 377
    std::cout << "Dec: " << std::dec << hex_val << std::endl;         // 255
    std::cout << "Bin: " << std::bitset<8>(hex_val) << std::endl;     // 11111111

    // Hex with uppercase and prefix
    std::cout << "Hex: 0x" << std::uppercase << std::hex << hex_val << std::endl;  // 0xFF

    // Parse hex string
    int parsed = std::stoi("CAF2", nullptr, 16);
    std::cout << "Parsed hex: " << std::dec << parsed << std::endl;   // 51954

    // Parse octal string
    int parsedOct = std::stoi("755", nullptr, 8);
    std::cout << "Parsed oct: " << parsedOct << std::endl;            // 493

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
        // Hex literal
        int hexVal = 0xFF;             // 255
        // Binary literal (C# 7.0+)
        int binVal = 0b1111_1111;      // 255
        // No octal literal in C#

        Console.WriteLine($"Decimal:     {hexVal}");                        // 255
        Console.WriteLine($"Hexadecimal: 0x{hexVal:X2}");                   // 0xFF
        Console.WriteLine($"Hexadecimal: {hexVal:x4}");                     // 00ff
        Console.WriteLine($"Binary:      {Convert.ToString(hexVal, 2).PadLeft(8, '0')}");  // 11111111
        Console.WriteLine($"Octal:       {Convert.ToString(hexVal, 8)}");   // 377

        // Parse from hex string
        int parsedHex = Convert.ToInt32("CAF2", 16);
        Console.WriteLine($"Parsed hex: {parsedHex}");   // 51954

        // Parse from octal string
        int parsedOct = Convert.ToInt32("755", 8);
        Console.WriteLine($"Parsed oct: {parsedOct}");   // 493

        // Hex is common for flags and masks
        int readWriteExecute = 0x07;  // binary: 0000 0111
        int readOnly         = 0x04;  // binary: 0000 0100
        // See [[Creating Bit Masks]] for mask patterns
    }
}
```

### JavaScript

```javascript
// Hex literal
let hexVal = 0xFF;             // 255
// Binary literal (ES2015+)
let binVal = 0b11111111;       // 255
// Octal literal (ES2015+, use 0o prefix)
let octVal = 0o377;            // 255

console.log(`Decimal:     ${hexVal}`);                         // 255
console.log(`Hexadecimal: 0x${hexVal.toString(16).toUpperCase()}`);  // 0xFF
console.log(`Binary:      ${hexVal.toString(2).padStart(8, '0')}`);  // 11111111
console.log(`Octal:       ${hexVal.toString(8)}`);             // 377

// Parse from hex string
let parsedHex = parseInt("CAF2", 16);
console.log(`Parsed hex: ${parsedHex}`);   // 51954

// Parse from octal string
let parsedOct = parseInt("755", 8);
console.log(`Parsed oct: ${parsedOct}`);   // 493

// Hex in real-world usage
let color = 0xFF8800;  // Orange in RGB
let red   = (color >> 16) & 0xFF;  // 0xFF = 255
let green = (color >> 8)  & 0xFF;  // 0x88 = 136
let blue  = color & 0xFF;          // 0x00 = 0
console.log(`RGB: (${red}, ${green}, ${blue})`);  // RGB: (255, 136, 0)
```

```ad-warning
title: Legacy octal in JavaScript
In non-strict mode, JavaScript still supports the old `0`-prefix octal syntax (e.g., `0377` = 255). In strict mode (`"use strict"`), this throws a syntax error. Always use `0o` for octal in modern code.
```

---

## Quick Conversion Cheat Sheet

```
Decimal  Binary       Hex    Octal
------  ----------   ----   -----
  0     0000 0000    0x00   000
  10    0000 1010    0x0A   012
  42    0010 1010    0x2A   052
  127   0111 1111    0x7F   177
  128   1000 0000    0x80   200
  200   1100 1000    0xC8   310
  255   1111 1111    0xFF   377
  256   1 0000 0000  0x100  400
```

---

## Related Notes

- [[Binary Number System]] — foundational binary concepts
- [[Bits Bytes and Words]] — how bits group into larger units
- [[Creating Bit Masks]] — practical use of hex for defining masks
- [[Color and Graphics]] — hex color codes in practice
