---
tags:
  - bit-manipulation
  - bit-mask
  - extraction
  - combining
---

## Combining and Extracting with Masks

Where [[Creating Bit Masks]] teaches how to build masks and [[Check Set and Clear Bits]] operates on single bits, this note scales those ideas to **multi-bit fields** -- groups of consecutive bits that encode a small value inside a larger integer. Packing several fields into one integer is the foundation of binary file formats, network protocols, GPU color values, and hardware register programming.

The two core operations are **extraction** (reading a field out of a packed value) and **insertion** (writing a field back in). Every technique here combines the [[AND Operator]], [[OR Operator]], [[NOT Operator]], [[Left Shift]], and [[Right Shift]] in predictable patterns.

---

## Extracting a Bit Field

To read a multi-bit field from a packed integer, you need two things: a **shift** to move the field down to bit 0, and a **mask** to strip away everything outside the field.

**Pattern:** `(value >> shift) & mask`

An equivalent form shifts the mask up instead: `(value & (mask << shift)) >> shift`. Both produce the same result; the first form is more common because the mask stays small.

```
value     = 0b_1101_0110_1010_0011
                     ^^^^
                     target field (bits 8-11)

Step 1: Shift right by 8
shifted   = 0b_0000_0000_1101_0110

Step 2: Mask with 0xF (4 bits wide)
            0b_0000_0000_1101_0110
          & 0b_0000_0000_0000_1111   (0xF)
          ---------------------------
result    = 0b_0000_0000_0000_0110   = 6
```

The mask width must match the field width. A 4-bit field uses `0xF` (binary `1111`), a 3-bit field uses `0x7` (binary `111`), and so on. You can compute the mask as `(1 << width) - 1`.

```ad-tip
title: Mask Width Formula
For a field that is **w** bits wide, the extraction mask is `(1 << w) - 1`. For 4 bits: `(1 << 4) - 1 = 15 = 0xF`. For 8 bits: `(1 << 8) - 1 = 255 = 0xFF`.
```

---

## Inserting / Replacing a Bit Field

To write a new value into a specific field without disturbing the surrounding bits, you perform three steps: **clear** the old field, **position** the new value, and **merge** them.

**Pattern:** `(value & ~mask) | ((newBits << shift) & mask)`

The `& mask` on the new bits is a safety guard -- it prevents `newBits` from being too wide and corrupting adjacent fields. If you trust the caller to pass valid values, you can omit it, but defensive code keeps it.

```
value    = 0b_1101_0110_1010_0011
Goal: replace bits 8-11 with 0xC (1100)

Step 1: Build the positioned mask
  mask   = 0xF << 8  = 0b_0000_1111_0000_0000

Step 2: Clear the target field
  ~mask  =             0b_1111_0000_1111_1111
  value & ~mask      = 0b_1101_0000_1010_0011
                             ^^^^
                             field is now 0000

Step 3: Position the new value
  newBits << 8       = 0b_0000_1100_0000_0000

Step 4: Merge with OR
  0b_1101_0000_1010_0011
| 0b_0000_1100_0000_0000
--------------------------
  0b_1101_1100_1010_0011
                ^^^^
                field is now 1100 = 0xC
```

This is exactly the same clear-then-set pattern from [[Check Set and Clear Bits]], expanded from one bit to a multi-bit field.

---

## Combining Multiple Fields into a Packed Value

When building a packed value from scratch (not modifying an existing one), you do not need the clear step. Simply shift each field into position and [[OR Operator|OR]] them together.

**Pattern:** `(field1 << shift1) | (field2 << shift2) | (field3 << shift3)`

Each field occupies a non-overlapping range of bits. The shift for each field equals the bit index of its lowest bit.

```
Suppose three fields:
  A = 3 bits (bits 5-7)
  B = 2 bits (bits 3-4)
  C = 3 bits (bits 0-2)

  A = 0b101, B = 0b11, C = 0b010

  (A << 5)  = 0b_1010_0000
  (B << 3)  = 0b_0001_1000
  (C << 0)  = 0b_0000_0010

  OR them:
    0b_1010_0000
  | 0b_0001_1000
  | 0b_0000_0010
  ---------------
    0b_1011_1010   = 0xBA = 186
```

```ad-warning
title: Overlapping Fields
If two fields share the same bit positions, OR merges their 1-bits rather than overwriting. You get corrupted data with no error. Always verify your shift offsets leave no overlaps and no gaps (unless gaps are intentional padding).
```

---

## Real-World Example 1: Packing RGB Color

A 24-bit RGB color stores three 8-bit channels in a single integer:

```
Bit: 23       16 15        8 7         0
    +----------+----------+----------+
    | Red (8b) |Green (8b)| Blue (8b)|
    +----------+----------+----------+
```

Each channel ranges from 0-255 (`0x00` to `0xFF`).

**Packing:**
```
  r = 0xAD, g = 0x45, b = 0xF7

  (r << 16) = 0x_AD_00_00   = 0b_1010_1101_0000_0000_0000_0000
  (g << 8)  = 0x_00_45_00   = 0b_0000_0000_0100_0101_0000_0000
  (b << 0)  = 0x_00_00_F7   = 0b_0000_0000_0000_0000_1111_0111

  OR them together:
  color     = 0x_AD_45_F7   = 0b_1010_1101_0100_0101_1111_0111
```

**Unpacking:**
```
  color = 0xAD45F7

  Red:   (color >> 16) & 0xFF  = 0xAD = 173
  Green: (color >>  8) & 0xFF  = 0x45 =  69
  Blue:  (color >>  0) & 0xFF  = 0xF7 = 247
```

### C\#

```csharp
public static class RgbPacker
{
    public static int Pack(byte r, byte g, byte b)
    {
        return (r << 16) | (g << 8) | b;
    }

    public static byte ExtractRed(int color)   => (byte)((color >> 16) & 0xFF);
    public static byte ExtractGreen(int color) => (byte)((color >>  8) & 0xFF);
    public static byte ExtractBlue(int color)  => (byte)( color        & 0xFF);
}

// Usage
int color = RgbPacker.Pack(173, 69, 247);   // 0xAD45F7
Console.WriteLine($"Packed: 0x{color:X6}"); // Packed: 0xAD45F7

byte r = RgbPacker.ExtractRed(color);       // 173
byte g = RgbPacker.ExtractGreen(color);     // 69
byte b = RgbPacker.ExtractBlue(color);      // 247
Console.WriteLine($"R={r}, G={g}, B={b}");
```

### C++

```cpp
#include <iostream>
#include <cstdint>
#include <iomanip>

uint32_t packRgb(uint8_t r, uint8_t g, uint8_t b) {
    return (static_cast<uint32_t>(r) << 16)
         | (static_cast<uint32_t>(g) << 8)
         | b;
}

uint8_t extractRed(uint32_t color)   { return (color >> 16) & 0xFF; }
uint8_t extractGreen(uint32_t color) { return (color >>  8) & 0xFF; }
uint8_t extractBlue(uint32_t color)  { return  color        & 0xFF; }

int main() {
    uint32_t color = packRgb(173, 69, 247);
    std::cout << "Packed: 0x" << std::hex << std::setfill('0')
              << std::setw(6) << color << "\n";   // 0xad45f7

    std::cout << std::dec;
    std::cout << "R=" << (int)extractRed(color)   << ", "
              << "G=" << (int)extractGreen(color) << ", "
              << "B=" << (int)extractBlue(color)  << "\n";
    return 0;
}
```

```ad-tip
title: Why static_cast in C++?
`uint8_t` is typically `unsigned char`. Without the cast, `r << 16` promotes `r` to `int` (which is signed). On most platforms this works fine for small values, but for values with the high bit set on a system where `int` is 16-bit, it would overflow. The explicit cast to `uint32_t` is the safe, portable habit.
```

### JavaScript

```javascript
function packRgb(r, g, b) {
    return (r << 16) | (g << 8) | b;
}

function extractRed(color)   { return (color >>> 16) & 0xFF; }
function extractGreen(color) { return (color >>>  8) & 0xFF; }
function extractBlue(color)  { return  color         & 0xFF; }

// Usage
const color = packRgb(173, 69, 247);
console.log("Packed: 0x" + color.toString(16).padStart(6, '0'));
// Packed: 0xad45f7

console.log(`R=${extractRed(color)}, G=${extractGreen(color)}, B=${extractBlue(color)}`);
// R=173, G=69, B=247
```

```ad-warning
title: JavaScript >>> vs >>
Use `>>>` (unsigned right shift) when extracting from values that might have bit 31 set (e.g., ARGB with alpha in bits 24-31). The standard `>>` sign-extends, which fills the upper bits with 1s if bit 31 is set, producing negative numbers. `>>>` always fills with 0s.
```

See also: [[Color and Graphics]]

---

## Real-World Example 2: Packing a Date (DOS Format)

The FAT file system uses a 16-bit packed date format. This is the same format used in ZIP file headers and was introduced in MS-DOS.

```
Bit: 15         9 8      5 4     0
    +------------+--------+------+
    | Year (7b)  |Mon (4b)|Day(5)|
    +------------+--------+------+

Year: 7 bits = values 0-127, added to base year 1980  (range 1980-2107)
Month: 4 bits = values 1-12
Day: 5 bits = values 1-31
```

**Packing (example: August 19, 2026):**
```
  year_offset = 2026 - 1980 = 46  = 0b_010_1110
  month       = 8                 = 0b_1000
  day         = 19                = 0b_1_0011

  (year_offset << 9)  = 0b_0101_1100_0000_0000
  (month << 5)        = 0b_0000_0001_0000_0000
  (day << 0)          = 0b_0000_0000_0001_0011

  OR together:
    0b_0101_1100_0000_0000
  | 0b_0000_0001_0000_0000
  | 0b_0000_0000_0001_0011
  -------------------------
    0b_0101_1101_0001_0011  = 0x5D13
```

**Unpacking:**
```
  packed = 0x5D13 = 0b_0101_1101_0001_0011

  Year offset: (packed >> 9) & 0x7F  = 0b_010_1110  = 46   -> 1980 + 46 = 2026
  Month:       (packed >> 5) & 0x0F  = 0b_1000      = 8    -> August
  Day:         (packed >> 0) & 0x1F  = 0b_1_0011    = 19
```

### C\#

```csharp
public static class DosDate
{
    public static ushort Pack(int year, int month, int day)
    {
        int yearOffset = year - 1980;
        return (ushort)(((yearOffset & 0x7F) << 9)
                      | ((month & 0x0F) << 5)
                      | (day & 0x1F));
    }

    public static (int year, int month, int day) Unpack(ushort packed)
    {
        int year  = ((packed >> 9) & 0x7F) + 1980;
        int month = (packed >> 5) & 0x0F;
        int day   = packed & 0x1F;
        return (year, month, day);
    }
}

// Usage
ushort packed = DosDate.Pack(2026, 8, 19);
Console.WriteLine($"Packed: 0x{packed:X4}");          // 0x5D13

var (y, m, d) = DosDate.Unpack(packed);
Console.WriteLine($"Unpacked: {y}-{m:D2}-{d:D2}");   // 2026-08-19
```

### C++

```cpp
#include <iostream>
#include <cstdint>
#include <iomanip>

uint16_t packDosDate(int year, int month, int day) {
    int yearOffset = year - 1980;
    return static_cast<uint16_t>(
        ((yearOffset & 0x7F) << 9)
      | ((month & 0x0F) << 5)
      | (day & 0x1F)
    );
}

struct Date { int year, month, day; };

Date unpackDosDate(uint16_t packed) {
    return {
        ((packed >> 9) & 0x7F) + 1980,
        (packed >> 5) & 0x0F,
        packed & 0x1F
    };
}

int main() {
    uint16_t packed = packDosDate(2026, 8, 19);
    std::cout << "Packed: 0x" << std::hex << std::setfill('0')
              << std::setw(4) << packed << "\n";

    Date d = unpackDosDate(packed);
    std::cout << std::dec << "Unpacked: "
              << d.year << "-"
              << std::setw(2) << std::setfill('0') << d.month << "-"
              << std::setw(2) << d.day << "\n";
    return 0;
}
```

### JavaScript

```javascript
function packDosDate(year, month, day) {
    const yearOffset = year - 1980;
    return ((yearOffset & 0x7F) << 9)
         | ((month & 0x0F) << 5)
         | (day & 0x1F);
}

function unpackDosDate(packed) {
    return {
        year:  ((packed >>> 9) & 0x7F) + 1980,
        month: (packed >>> 5) & 0x0F,
        day:   packed & 0x1F
    };
}

// Usage
const packed = packDosDate(2026, 8, 19);
console.log("Packed: 0x" + packed.toString(16).padStart(4, '0'));
// Packed: 0x5d13

const { year, month, day } = unpackDosDate(packed);
console.log(`Unpacked: ${year}-${String(month).padStart(2,'0')}-${String(day).padStart(2,'0')}`);
// Unpacked: 2026-08-19
```

```ad-note
title: DOS Date in the Wild
This exact format appears in ZIP file local headers (bytes 12-13), FAT12/FAT16/FAT32 directory entries, and Windows PE headers. When you parse binary files in any of these formats, you are doing exactly this bit field extraction.
```

---

## Signed Field Extraction

All the examples above treat extracted fields as **unsigned** values. When a bit field represents a **signed** value (two's complement), naive extraction gives the wrong result for negative numbers because the sign bit is not extended.

Consider a 4-bit signed field (range -8 to +7). The value `-3` is stored as `0b1101` in two's complement.

```
Packed value: 0b_xxxx_1101_xxxx_xxxx  (4-bit field at bits 8-11)

Naive extraction:
  (value >> 8) & 0xF  = 0b_0000_1101 = 13   (WRONG, should be -3)
```

The problem: the mask strips the sign bit's significance. The extracted `0b1101` is interpreted as unsigned 13 instead of signed -3.

**Fix: Sign extension**

After extracting, check the sign bit and extend it:

```
Step 1: Extract normally
  raw = (value >> 8) & 0xF     // = 13 (0b1101)

Step 2: Check sign bit (bit 3 for a 4-bit field)
  sign_bit = raw & (1 << 3)   // = 8 (nonzero -> negative)

Step 3: Extend the sign
  if sign_bit != 0:
      result = raw | ~0xF     // fill upper bits with 1s
                              // = 0xFFFF_FFFD = -3

Alternative one-liner (arithmetic):
  result = raw - (raw & (1 << (width - 1))) * 2

Shift-based trick:
  result = (raw << (32 - width)) >> (32 - width)   // arithmetic shift
```

The shift-based trick works by moving the sign bit to position 31 (the int sign bit) and then using arithmetic right shift (`>>` in C#/C++/Java) to replicate it back down.

```ad-warning
title: Sign Extension Gotchas
1. **C++ `>>` on signed types is implementation-defined.** Most compilers do arithmetic shift (sign-extending), but the standard does not guarantee it until C++20. Use explicit sign extension to be safe.
2. **JavaScript `>>` is arithmetic** (sign-extends), while `>>>` is logical (zero-fills). Use `>>` for signed extraction.
3. Forgetting sign extension is a common source of bugs when parsing formats like EXIF metadata, audio samples, or sensor data where fields are signed but narrow.
```

---

## C++ Struct Bitfields vs Manual Bit Manipulation

C++ offers a language-level alternative for bit-packed structures: **bitfield members**. They let the compiler generate the shifting and masking automatically.

```cpp
// Compiler-managed bitfield
struct DosDateBitfield {
    uint16_t day   : 5;
    uint16_t month : 4;
    uint16_t year  : 7;   // offset from 1980
};

// Usage
DosDateBitfield d;
d.year  = 2026 - 1980;
d.month = 8;
d.day   = 19;

// Access fields like normal struct members
std::cout << d.year + 1980 << "-" << d.month << "-" << d.day << "\n";
```

```ad-warning
title: Bitfield Portability Problems
Struct bitfields are **not portable** across compilers or architectures:
- The **bit order** (MSB-first or LSB-first within a byte) is implementation-defined
- **Padding** and **alignment** between fields is implementation-defined
- You **cannot** reliably `memcpy` a bitfield struct into a `uint16_t` and expect the bits to match the manual layout above

For binary file formats, network protocols, or cross-platform code, **always use manual bit manipulation**. Struct bitfields are only safe for compiler-internal usage on a single platform (e.g., hardware register definitions provided by a chip vendor's header file).
```

---

## Summary

| Operation            | Pattern                                          | Key Operators              |
| -------------------- | ------------------------------------------------ | -------------------------- |
| Extract field        | `(value >> shift) & mask`                        | [[Right Shift]], [[AND Operator]] |
| Insert/replace field | `(value & ~posMask) \| ((new << shift) & posMask)` | [[NOT Operator]], [[AND Operator]], [[OR Operator]] |
| Combine from scratch | `(a << s1) \| (b << s2) \| (c << s3)`           | [[Left Shift]], [[OR Operator]] |
| Mask for width w     | `(1 << w) - 1`                                  | [[Left Shift]]             |
| Sign-extend          | `(raw << (32-w)) >> (32-w)`                      | Arithmetic [[Right Shift]] |

---

## See Also

- [[Creating Bit Masks]] -- building the masks used in these operations
- [[AND Operator]] -- the operator behind field extraction
- [[OR Operator]] -- the operator behind field combination
- [[NOT Operator]] -- used to invert masks for field clearing
- [[Left Shift]] -- positions new values into their target field
- [[Right Shift]] -- moves target fields down to bit 0 for reading
- [[Color and Graphics]] -- real-world application of RGB packing
- [[Check Set and Clear Bits]] -- single-bit versions of these operations
