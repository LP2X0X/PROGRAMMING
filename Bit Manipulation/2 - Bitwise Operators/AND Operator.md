---
title: AND Operator
date: 2026-08-19
tags:
  - bit-manipulation
  - operator
  - bitwise-and
aliases:
  - Bitwise AND
  - "&"
  - AND gate
---

## AND Operator (`&`)

The **AND operator** (`&`) is one of the most fundamental [[Binary Number System|bitwise operators]]. It compares each pair of corresponding bits from two operands and produces a `1` in the result **only if both bits are `1`**. In every other case, the result bit is `0`. This behavior mirrors the logical AND gate in digital electronics.

The AND operator is the workhorse behind **bit masking**, **flag checking**, **bit clearing**, and **field extraction** -- operations that appear constantly in systems programming, embedded development, networking, graphics, and performance-critical code.

---

## Table of Contents

- [[#Truth Table]]
- [[#Bit-by-Bit Operation Visual]]
- [[#Syntax in C++, C#, and JavaScript]]
- [[#Practical Uses]]
  - [[#Masking Bits]]
  - [[#Checking if a Bit Is Set]]
  - [[#Clearing Bits]]
  - [[#Extracting Bit Fields]]
- [[#Full Code Examples]]
  - [[#C++ Example]]
  - [[#C# Example]]
  - [[#JavaScript Example]]
- [[#Relationship to Other Operators]]
- [[#Common Pitfalls]]
- [[#Section Summary — Full Note]]

---

## Truth Table

The AND truth table is the foundation for understanding every use case below. Memorize this -- it is the single rule that governs the entire operator.

| A   | B   | A & B |
| --- | --- | ----- |
| 0   | 0   | 0     |
| 0   | 1   | 0     |
| 1   | 0   | 0     |
| 1   | 1   | 1     |

The key insight: **AND only outputs `1` when both inputs are `1`**. Think of it as a gate that requires *both* conditions to pass.

```ad-tip
title: Memory Aid
AND is the "strict" operator. Both bits must agree on `1` for the result to be `1`. Compare with [[OR Operator]] (either bit can be `1`) and [[XOR Operator]] (exactly one bit must be `1`).
```

```ad-note
title: Analogy
Think of AND like two light switches wired in series. The light (result) only turns on if *both* switches are flipped to ON. If either switch is OFF, the light stays OFF.
```

---

## Bit-by-Bit Operation Visual

Below is an ASCII diagram showing how AND operates on two 8-bit values. The operation is applied **independently to each column** (bit position):

```
  Example: 0b11001010 & 0b10101100

  Bit Position:   7   6   5   4   3   2   1   0
                +---+---+---+---+---+---+---+---+
  Operand A:    | 1 | 1 | 0 | 0 | 1 | 0 | 1 | 0 |   = 0xCA (202)
                +---+---+---+---+---+---+---+---+
                  &   &   &   &   &   &   &   &
                +---+---+---+---+---+---+---+---+
  Operand B:    | 1 | 0 | 1 | 0 | 1 | 1 | 0 | 0 |   = 0xAC (172)
                +---+---+---+---+---+---+---+---+
                  =   =   =   =   =   =   =   =
                +---+---+---+---+---+---+---+---+
  Result:       | 1 | 0 | 0 | 0 | 1 | 0 | 0 | 0 |   = 0x88 (136)
                +---+---+---+---+---+---+---+---+
```

Walk through each column:
- **Bit 7**: `1 & 1 = 1`
- **Bit 6**: `1 & 0 = 0`
- **Bit 5**: `0 & 1 = 0`
- **Bit 4**: `0 & 0 = 0`
- **Bit 3**: `1 & 1 = 1`
- **Bit 2**: `0 & 1 = 0`
- **Bit 1**: `1 & 0 = 0`
- **Bit 0**: `0 & 0 = 0`

```ad-tip
title: Second Visual -- Masking in Action
Here is a common real-world pattern: extracting the lower nibble (lower 4 bits) from a byte using a mask of `0x0F`.

    Value:    1 0 1 1 0 1 1 0   (0xB6 = 182)
    Mask:     0 0 0 0 1 1 1 1   (0x0F =  15)
              ─ ─ ─ ─ ─ ─ ─ ─
    Result:   0 0 0 0 0 1 1 0   (0x06 =   6)

The mask "lets through" only the bits where the mask has a `1`, and "blocks" (zeroes out) all bits where the mask has a `0`. This is the essence of [[Creating Bit Masks|bit masking]].
```

---

## Syntax in C++, C#, and JavaScript

The AND operator uses the same symbol (`&`) in all three languages. However, there are subtle differences in types and behavior.

### C++

```cpp
int result = a & b;          // bitwise AND on integers
unsigned int r = x & mask;   // common with unsigned types
auto flags = status & 0xFF;  // mask with hex literal
```

- Works on all integer types: `int`, `unsigned int`, `long`, `char`, `uint8_t`, etc.
- On [[Signed and Unsigned Integers|signed types]], be aware that the sign bit participates in the AND operation. Prefer `unsigned` types for bitwise work.

### C#

```csharp
int result = a & b;          // bitwise AND on int
uint r = x & mask;           // common with uint
byte lower = (byte)(val & 0xFF); // explicit cast needed for smaller types
```

- Works on `int`, `uint`, `long`, `ulong`, `byte`, `sbyte`, `short`, `ushort`.
- C# requires explicit casts when assigning results back to smaller types like `byte`.
- Also works on `enum` types decorated with `[Flags]` -- essential for [[Flag Enums and Bit Flags]].

### JavaScript

```javascript
let result = a & b;          // bitwise AND
let masked = value & 0xFF;   // mask lower byte
let check = flags & (1 << n); // check bit n
```

```ad-warning
title: JavaScript Truncation to 32 Bits
JavaScript converts operands to **signed 32-bit integers** before performing any bitwise operation. This means:
- Numbers larger than `2^31 - 1` will lose precision or produce unexpected results.
- Floating-point values are truncated to integers first.
- If you need 64-bit bitwise operations, use `BigInt` instead: `const r = 0xFFFFFFFFFFn & valuen;`
```

```ad-note
title: Don't Confuse & with &&
In all three languages, `&` is the **bitwise** AND while `&&` is the **logical** AND (short-circuit boolean). Using `&` where you mean `&&` will compile but produce wrong results when operands are not 0 or 1. Conversely, using `&&` for bitwise work will not give you bit-level manipulation at all.
```

---

## Practical Uses

### Masking Bits

**Bit masking** is the most common use of AND. A mask is a value with `1`s in the bit positions you want to keep and `0`s everywhere else. ANDing a value with a mask "extracts" only those bits.

```
value & mask = only the bits where mask has 1
```

See [[Creating Bit Masks]] for how to construct masks for any situation.

**Example: Extract the green channel from an RGB color**

An RGB pixel stored as `0xRRGGBB`:

```cpp
uint32_t pixel = 0x3FA86C;
uint32_t green = (pixel & 0x00FF00) >> 8;  // green = 0xA8 = 168
```

The mask `0x00FF00` has `1`s only in bits 8-15 (the green byte), so AND zeroes out the red and blue channels, leaving only green. Then a [[Left Shift|right shift]] moves the result to the lowest byte.

### Checking if a Bit Is Set

To test whether bit `n` is set in a value, AND the value with a mask that has a `1` only at position `n`:

```
if (value & (1 << n)) {
    // bit n is set
}
```

Here, `1 << n` creates a mask with a single `1` at position `n` (see [[Left Shift]]). If the result is non-zero, the bit was set. If zero, the bit was clear.

This is the foundation of [[Check Set and Clear Bits|checking bits]] and is used extensively with [[Flag Enums and Bit Flags]].

```ad-tip
title: Why Non-Zero Means Set
When you AND a value with a single-bit mask, the result is either `0` (bit was not set) or `2^n` (the mask value itself, because that bit *was* set). Any non-zero value is truthy in C++, C#, and JavaScript, so it works directly in `if` conditions.
```

### Clearing Bits

To clear (set to `0`) specific bits, AND the value with the [[NOT Operator|bitwise complement]] of a mask:

```
value = value & ~mask;
```

The `~mask` inverts all bits of the mask: `1`s become `0`s and vice versa. ANDing with this inverted mask preserves all bits *except* those targeted by the original mask, which get forced to `0`.

**Example: Clear bit 3**

```
value = value & ~(1 << 3);
```

This is the complement of "setting" a bit (which uses [[OR Operator]]). See [[Check Set and Clear Bits]] for the full set/clear/toggle pattern.

### Extracting Bit Fields

Many hardware registers, file formats, and protocols pack multiple values into sub-ranges of bits within a single integer. To extract a field:

1. AND with a mask that covers the field's bit positions
2. Right-shift the result to align the field to bit 0

```
field = (value & fieldMask) >> fieldOffset;
```

**Example: Extract bits 4-6 from an 8-bit register**

```
  Register:   0 1 1 0 1 1 0 1   (0x6D)
  Mask:       0 1 1 1 0 0 0 0   (0x70) -- bits 4,5,6
              ─ ─ ─ ─ ─ ─ ─ ─
  AND result: 0 1 1 0 0 0 0 0   (0x60)
  >> 4:       0 0 0 0 0 1 1 0   (0x06 = 6)
```

The extracted field value is `6`. This pattern appears everywhere: CPU instruction decoding, parsing binary protocols, reading hardware status registers.

```ad-note
title: Section Summary — Practical Uses
- **Masking**: AND with a mask to isolate specific bits from a value
- **Checking**: AND with `(1 << n)` to test if bit n is set; result is non-zero if set
- **Clearing**: AND with `~mask` to force specific bits to zero
- **Extracting fields**: AND with a field mask then right-shift to get the field value
- AND never *sets* bits to 1 -- it can only preserve or clear them
```

---

## Full Code Examples

### C++ Example

```cpp
#include <cstdint>
#include <cstdio>

int main() {
    // 1. Masking -- extract lower nibble
    uint8_t value = 0xB6;  // 1011 0110
    uint8_t lowerNibble = value & 0x0F;  // 0000 0110 = 6
    printf("Lower nibble of 0x%02X: 0x%02X (%d)\n", value, lowerNibble, lowerNibble);

    // 2. Checking if a bit is set
    uint8_t flags = 0b10100101;
    for (int i = 7; i >= 0; --i) {
        bool isSet = (flags & (1 << i)) != 0;
        printf("Bit %d: %s\n", i, isSet ? "SET" : "CLEAR");
    }

    // 3. Clearing bits -- clear bits 2 and 3
    uint8_t reg = 0xFF;              // 1111 1111
    uint8_t mask = (1 << 2) | (1 << 3);  // 0000 1100
    reg = reg & ~mask;               // 1111 0011 = 0xF3
    printf("After clearing bits 2,3: 0x%02X\n", reg);

    // 4. Extracting a bit field -- bits 4-6
    uint8_t status = 0x6D;           // 0110 1101
    uint8_t field = (status & 0x70) >> 4;  // 0x70 = 0111 0000
    printf("Field (bits 4-6): %d\n", field);

    // 5. Real-world: subnet masking
    uint32_t ip   = 0xC0A80164;     // 192.168.1.100
    uint32_t mask2 = 0xFFFFFF00;    // 255.255.255.0  (/24)
    uint32_t network = ip & mask2;  // 192.168.1.0
    printf("Network address: %d.%d.%d.%d\n",
           (network >> 24) & 0xFF, (network >> 16) & 0xFF,
           (network >> 8) & 0xFF, network & 0xFF);

    return 0;
}
```

### C# Example

```csharp
using System;

class BitwiseAndDemo
{
    [Flags]
    enum Permissions
    {
        None    = 0,
        Read    = 1 << 0,  // 0001
        Write   = 1 << 1,  // 0010
        Execute = 1 << 2,  // 0100
        Delete  = 1 << 3,  // 1000
        All     = Read | Write | Execute | Delete
    }

    static void Main()
    {
        // 1. Masking -- extract lower nibble
        byte value = 0xB6;  // 1011 0110
        byte lowerNibble = (byte)(value & 0x0F);  // cast required for byte
        Console.WriteLine($"Lower nibble of 0x{value:X2}: 0x{lowerNibble:X2} ({lowerNibble})");

        // 2. Checking if a bit is set
        byte flags = 0b_1010_0101;
        for (int i = 7; i >= 0; i--)
        {
            bool isSet = (flags & (1 << i)) != 0;
            Console.WriteLine($"Bit {i}: {(isSet ? "SET" : "CLEAR")}");
        }

        // 3. Clearing bits -- clear bits 2 and 3
        byte reg = 0xFF;
        byte mask = (byte)((1 << 2) | (1 << 3));
        reg = (byte)(reg & ~mask);
        Console.WriteLine($"After clearing bits 2,3: 0x{reg:X2}");

        // 4. Extracting a bit field -- bits 4-6
        byte status = 0x6D;
        int field = (status & 0x70) >> 4;
        Console.WriteLine($"Field (bits 4-6): {field}");

        // 5. Real-world: Flag enum checking
        Permissions userPerms = Permissions.Read | Permissions.Write | Permissions.Execute;

        bool canWrite = (userPerms & Permissions.Write) == Permissions.Write;
        Console.WriteLine($"Can write: {canWrite}");

        // HasFlag is the idiomatic C# way (same as above under the hood)
        bool canDelete = userPerms.HasFlag(Permissions.Delete);
        Console.WriteLine($"Can delete: {canDelete}");

        // Remove a permission using AND + NOT
        Permissions restricted = userPerms & ~Permissions.Execute;
        Console.WriteLine($"After removing Execute: {restricted}");
    }
}
```

### JavaScript Example

```javascript
// 1. Masking -- extract lower nibble
const value = 0xB6;  // 1011 0110
const lowerNibble = value & 0x0F;  // 0000 0110 = 6
console.log(`Lower nibble of 0x${value.toString(16).toUpperCase()}: ` +
            `0x${lowerNibble.toString(16).toUpperCase()} (${lowerNibble})`);

// 2. Checking if a bit is set
const flags = 0b10100101;
for (let i = 7; i >= 0; i--) {
    const isSet = (flags & (1 << i)) !== 0;
    console.log(`Bit ${i}: ${isSet ? "SET" : "CLEAR"}`);
}

// 3. Clearing bits -- clear bits 2 and 3
let reg = 0xFF;
const mask = (1 << 2) | (1 << 3);
reg = reg & ~mask;
console.log(`After clearing bits 2,3: 0x${reg.toString(16).toUpperCase()}`);

// 4. Extracting a bit field -- bits 4-6
const status = 0x6D;
const field = (status & 0x70) >> 4;
console.log(`Field (bits 4-6): ${field}`);

// 5. Real-world: parsing a 32-bit RGBA color
const rgba = 0xFF8040CC;  // R=FF, G=80, B=40, A=CC
// NOTE: JavaScript bitwise ops use signed 32-bit, so large hex may be negative
const r = (rgba >>> 24) & 0xFF;  // >>> is unsigned right shift
const g = (rgba >>> 16) & 0xFF;
const b = (rgba >>> 8)  & 0xFF;
const a = rgba & 0xFF;
console.log(`R=${r} G=${g} B=${b} A=${a}`);

// 6. BigInt for values beyond 32 bits
const big = 0x1FFFFFFFFFFn;
const masked = big & 0xFFFFFFFFn;  // keep lower 32 bits
console.log(`BigInt masked: 0x${masked.toString(16)}`);
```

```ad-note
title: Section Summary — Code Examples
- All three languages use `&` for bitwise AND with nearly identical syntax
- C# requires explicit casts when assigning back to `byte` or other small types
- JavaScript truncates to signed 32-bit; use `>>>` for unsigned shift and `BigInt` for 64-bit
- The four core patterns (masking, checking, clearing, extracting) look the same across all languages
- Real-world examples include color parsing, flag enums, subnet masking, and register manipulation
```

---

## Relationship to Other Operators

Understanding AND in context with the other bitwise operators clarifies when to reach for each one:

| Operation     | Operator | Effect on Target Bits | Use Case                                        |
| ------------- | -------- | --------------------- | ----------------------------------------------- |
| AND           | `&`      | Preserve or clear     | Masking, checking, clearing, extracting          |
| [[OR Operator]]  | `\|`     | Preserve or set       | Setting bits, combining flags                    |
| [[XOR Operator]] | `^`      | Preserve or toggle    | Toggling bits, swapping values, checksums        |
| [[NOT Operator]] | `~`      | Invert all bits       | Creating inverted masks, [[Twos Complement\|two's complement]] |

Key relationships:
- **AND + NOT = Clear**: `value & ~mask` clears specific bits
- **OR = Set**: `value | mask` sets specific bits (see [[OR Operator]])
- **XOR = Toggle**: `value ^ mask` flips specific bits (see [[XOR Operator]])
- **AND + Shift = Extract**: `(value & mask) >> offset` extracts a bit field

These four operations form the complete toolkit for [[Check Set and Clear Bits|bit manipulation]].

---

## Common Pitfalls

```ad-warning
title: Pitfall 1 -- Confusing & with &&
`&` is bitwise AND. `&&` is logical AND (short-circuit). Using `&` in a boolean expression like `if (x > 0 & y > 0)` will work in C# and C++ but evaluates *both* sides (no short-circuit). In most cases you want `&&` for boolean logic. Reserve `&` for bit-level operations.
```

```ad-warning
title: Pitfall 2 -- Operator Precedence
In all three languages, bitwise `&` has **lower precedence** than comparison operators (`==`, `!=`, `<`, `>`). This means:

    if (flags & 0x04 == 0x04)   // WRONG: parsed as flags & (0x04 == 0x04)
    if ((flags & 0x04) == 0x04) // CORRECT: parentheses force AND first

Always use parentheses around bitwise expressions in conditions. This is one of the most common bugs in bitwise code.
```

```ad-warning
title: Pitfall 3 -- Sign Extension with Signed Types
When using AND on [[Signed and Unsigned Integers|signed integers]], the sign bit is just another bit. ANDing a negative number with a positive mask can produce unexpected results if you forget about [[Twos Complement|two's complement]] representation. Prefer unsigned types (`unsigned int`, `uint`, `>>>` in JS) for bitwise work.
```

```ad-warning
title: Pitfall 4 -- JavaScript 32-Bit Truncation
JavaScript silently converts all bitwise operands to signed 32-bit integers. If you AND a number larger than `0x7FFFFFFF`, the upper bits are lost. Use `BigInt` when working with values that exceed 32 bits.
```

```ad-tip
title: Subnet Masking
AND is how [[Networking and Subnet Masks|subnet masks]] work in networking. An IP address ANDed with a subnet mask yields the network address. For example:
- IP: `192.168.1.100` = `0xC0A80164`
- Mask: `255.255.255.0` = `0xFFFFFF00`
- Network: `192.168.1.0` = `0xC0A80100`

This is a direct application of the masking pattern.
```

---

## Section Summary -- Full Note

```ad-note
title: Comprehensive Summary
The AND operator (`&`) compares bits pairwise and outputs `1` only when both input bits are `1`. This single rule underpins four core patterns:

1. **Masking** -- AND with a mask to isolate specific bits; bits where the mask is `0` are zeroed out
2. **Checking** -- AND with `(1 << n)` to test if a specific bit is set; non-zero means set
3. **Clearing** -- AND with `~mask` (the NOT of a mask) to force specific bits to zero without disturbing others
4. **Extracting** -- AND with a field mask then right-shift to pull out a multi-bit field value

AND never sets bits to `1` -- it can only preserve existing `1`s or clear them to `0`. To set bits, use [[OR Operator]]. To toggle bits, use [[XOR Operator]].

Critical pitfalls: always parenthesize `&` in conditions (precedence is lower than `==`), never confuse `&` with `&&`, prefer unsigned types for bitwise work, and remember JavaScript truncates to 32-bit signed integers.

Real-world applications span color channel extraction, [[Flag Enums and Bit Flags|flag checking]], [[Networking and Subnet Masks|subnet masking]], hardware register manipulation, binary protocol parsing, and performance optimizations where bitwise checks replace arithmetic.
```

---

## Related Topics

- [[Creating Bit Masks]]
- [[Check Set and Clear Bits]]
- [[OR Operator]]
- [[XOR Operator]]
- [[NOT Operator]]
- [[Left Shift]]
- [[Binary Number System]]
- [[Flag Enums and Bit Flags]]
- [[Networking and Subnet Masks]]
- [[Twos Complement]]
- [[Signed and Unsigned Integers]]
