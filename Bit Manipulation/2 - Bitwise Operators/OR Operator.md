---
title: "OR Operator"
date: 2026-08-19
tags:
  - bit-manipulation
  - operator
  - bitwise-or
aliases:
  - Bitwise OR
  - OR gate
  - Inclusive OR
---

## OR Operator (`|`)

The **bitwise OR operator** (`|`) compares each bit of two operands and produces a `1` in the result if **either or both** corresponding bits are `1`. It is one of the fundamental [[AND Operator|bitwise operators]] alongside [[AND Operator]], [[XOR Operator]], and [[NOT Operator]]. OR is the go-to operation for **setting bits** and **combining flags** in the [[Binary Number System]].

---

## Truth Table

The OR operator follows a simple rule: the result bit is `1` if **at least one** of the input bits is `1`.

| A | B | A \| B |
|---|---|--------|
| 0 | 0 |   0    |
| 0 | 1 |   1    |
| 1 | 0 |   1    |
| 1 | 1 |   1    |

```ad-tip
title: Memory Aid
OR is the "optimistic" operator -- if *either* input says yes (`1`), the output says yes. The only way to get `0` is if *both* inputs are `0`. Contrast this with [[AND Operator]], which is "pessimistic" -- both must be `1` to produce `1`.
```

---

## Bit-by-Bit Operation Visual

Below is an ASCII diagram showing how OR operates on two 8-bit values, comparing each column (bit position) independently:

```
  Example: 0b11001010 | 0b10110110

  Bit Position:   7   6   5   4   3   2   1   0
                +---+---+---+---+---+---+---+---+
  Operand A:    | 1 | 1 | 0 | 0 | 1 | 0 | 1 | 0 |   = 0xCA (202)
                +---+---+---+---+---+---+---+---+
  Operand B:    | 1 | 0 | 1 | 1 | 0 | 1 | 1 | 0 |   = 0xB6 (182)
                +---+---+---+---+---+---+---+---+
      OR  |     | | | | | | | | | | | | | | | | |
                +---+---+---+---+---+---+---+---+
  Result:       | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 0 |   = 0xFE (254)
                +---+---+---+---+---+---+---+---+

  Column-by-column:
    Bit 7: 1 | 1 = 1
    Bit 6: 1 | 0 = 1
    Bit 5: 0 | 1 = 1
    Bit 4: 0 | 1 = 1
    Bit 3: 1 | 0 = 1
    Bit 2: 0 | 1 = 1
    Bit 1: 1 | 1 = 1
    Bit 0: 0 | 0 = 0    <-- only column where BOTH are 0
```

```ad-note
title: Key Observation
Notice that the result has *more* `1` bits than either operand alone. OR can only **set** bits to `1`; it can never turn a `1` into a `0`. This is the fundamental property that makes OR the tool of choice for turning bits ON. See [[Check Set and Clear Bits]] for the full set/clear/toggle toolkit.
```

---

## Syntax

The bitwise OR operator uses the pipe character `|` in all three languages. Do not confuse it with the **logical OR** operator (`||`), which operates on boolean values and short-circuits.

### C++

```cpp
int result = a | b;       // bitwise OR
result |= mask;           // compound assignment (OR and assign)
```

### C#

```csharp
int result = a | b;       // bitwise OR on integers
result |= mask;           // compound assignment

// Also works on enum types decorated with [Flags]
FileAccess access = FileAccess.Read | FileAccess.Write;
```

### JavaScript

```javascript
let result = a | b;       // bitwise OR (converts operands to 32-bit signed ints)
result |= mask;           // compound assignment
```

```ad-warning
title: JavaScript Truncation
In JavaScript, the `|` operator converts both operands to **32-bit signed integers** before performing the operation. This means values larger than `2^31 - 1` or with fractional parts will be truncated. For bitwise work beyond 32 bits, use `BigInt` values with `BigInt(a) | BigInt(b)`.
```

---

## Practical Uses

### 1. Setting Bits (Turning Specific Bits ON)

The most fundamental use of OR is to **set** (turn ON) one or more bits in a value without affecting the other bits. You OR the value with a **mask** where only the target bit positions are `1`.

```
  Before:     0 1 0 0 1 0 0 0     = 0x48
  Mask:       0 0 0 0 0 1 0 0     = 0x04  (bit 2)
              ─────────────────
  After OR:   0 1 0 0 1 1 0 0     = 0x4C
                          ^
                     bit 2 is now SET
```

This is the inverse of using [[AND Operator]] with an inverted mask to **clear** bits. See [[Check Set and Clear Bits]] for the complete pattern and [[Creating Bit Masks]] for how to construct masks using [[Left Shift]].

```ad-tip
title: Idempotent Operation
ORing a bit that is already `1` leaves it as `1`. This means setting a bit with OR is **idempotent** -- you can safely set it multiple times without side effects. You never need to check whether a bit is already set before ORing it on.
```

### 2. Combining Flags

The most common real-world use of OR is **combining flags** in flag enumerations. Each flag is a power of two (a single bit), and OR merges them into a single value that represents the combination. See [[Flag Enums and Bit Flags]] for a deep dive.

```csharp
// C# -- combining file access flags
FileAccess access = FileAccess.Read | FileAccess.Write;

// C# -- combining custom permission flags
[Flags]
enum Permissions
{
    None    = 0,       // 0b0000
    Read    = 1,       // 0b0001
    Write   = 2,       // 0b0010
    Execute = 4,       // 0b0100
    Delete  = 8,       // 0b1000
    All     = 15       // 0b1111
}

Permissions userPerms = Permissions.Read | Permissions.Write | Permissions.Execute;
// userPerms = 0b0111 = 7
```

See [[Permissions and Access Flags]] for how this pattern is used in operating systems, file systems, and application security.

### 3. Merging Bit Fields

OR is used to **pack** multiple smaller values into a single larger integer. Each sub-value is shifted into its position with [[Left Shift]] and then ORed together. See [[Combining and Extracting with Masks]] for the extraction side using [[AND Operator]].

```
  Packing an RGB color (8 bits each) into a 24-bit integer:

  R = 0xFF (255)    shifted left 16:   0xFF0000
  G = 0x80 (128)    shifted left  8:   0x008000
  B = 0x3C  (60)    shifted left  0:   0x00003C
                                       ────────
  OR all together:                     0xFF803C

  color = (R << 16) | (G << 8) | B
```

---

## Full Code Examples

### C++

```cpp
#include <iostream>
#include <cstdint>
#include <bitset>

int main() {
    // --- Basic OR ---
    uint8_t a = 0b11001010;  // 202
    uint8_t b = 0b10110110;  // 182
    uint8_t result = a | b;  // 0b11111110 = 254

    std::cout << "a      = " << std::bitset<8>(a) << " (" << (int)a << ")\n";
    std::cout << "b      = " << std::bitset<8>(b) << " (" << (int)b << ")\n";
    std::cout << "a | b  = " << std::bitset<8>(result) << " (" << (int)result << ")\n\n";

    // --- Setting a specific bit ---
    uint8_t flags = 0b00001000;  // bit 3 is set
    uint8_t mask  = 0b00000100;  // we want to also set bit 2
    flags |= mask;
    // flags is now 0b00001100 -- bits 3 and 2 are set
    std::cout << "After setting bit 2: " << std::bitset<8>(flags) << "\n\n";

    // --- Combining flags (simulating permissions) ---
    enum Permission : uint8_t {
        PERM_NONE    = 0,        // 0b00000000
        PERM_READ    = 1 << 0,   // 0b00000001
        PERM_WRITE   = 1 << 1,   // 0b00000010
        PERM_EXECUTE = 1 << 2,   // 0b00000100
    };

    uint8_t userPerms = PERM_READ | PERM_WRITE | PERM_EXECUTE;
    std::cout << "User permissions: " << std::bitset<8>(userPerms) << " (" << (int)userPerms << ")\n";

    // Check if a permission is set (uses AND -- see AND Operator note)
    if (userPerms & PERM_WRITE) {
        std::cout << "User has WRITE permission\n";
    }

    // --- Packing RGB into a 32-bit integer ---
    uint8_t r = 0xFF, g = 0x80, bVal = 0x3C;
    uint32_t color = (static_cast<uint32_t>(r) << 16)
                   | (static_cast<uint32_t>(g) << 8)
                   | bVal;
    std::cout << "\nPacked RGB color: 0x" << std::hex << color << std::dec << "\n";

    return 0;
}
```

### C#

```csharp
using System;

class OrOperatorDemo
{
    [Flags]
    enum FilePermission
    {
        None    = 0,       // 0b0000_0000
        Read    = 1 << 0,  // 0b0000_0001
        Write   = 1 << 1,  // 0b0000_0010
        Execute = 1 << 2,  // 0b0000_0100
        Delete  = 1 << 3,  // 0b0000_1000
        All     = Read | Write | Execute | Delete
    }

    static void Main()
    {
        // --- Basic OR ---
        byte a = 0b_1100_1010;  // 202
        byte b = 0b_1011_0110;  // 182
        int result = a | b;     // 0b_1111_1110 = 254

        Console.WriteLine($"a     = {Convert.ToString(a, 2).PadLeft(8, '0')} ({a})");
        Console.WriteLine($"b     = {Convert.ToString(b, 2).PadLeft(8, '0')} ({b})");
        Console.WriteLine($"a | b = {Convert.ToString(result, 2).PadLeft(8, '0')} ({result})");
        Console.WriteLine();

        // --- Setting a specific bit ---
        int flags = 0b_0000_1000;  // bit 3 set
        int mask  = 0b_0000_0100;  // target bit 2
        flags |= mask;
        Console.WriteLine($"After setting bit 2: {Convert.ToString(flags, 2).PadLeft(8, '0')}");
        Console.WriteLine();

        // --- Combining flag enums ---
        FilePermission perms = FilePermission.Read | FilePermission.Write;
        Console.WriteLine($"Permissions: {perms}");  // "Read, Write"

        // Add Execute permission later
        perms |= FilePermission.Execute;
        Console.WriteLine($"After adding Execute: {perms}");  // "Read, Write, Execute"

        // Check permission (uses AND)
        if ((perms & FilePermission.Write) != 0)
        {
            Console.WriteLine("User has Write permission");
        }
        Console.WriteLine();

        // --- Packing RGB ---
        byte r = 0xFF, g = 0x80, bVal = 0x3C;
        int color = (r << 16) | (g << 8) | bVal;
        Console.WriteLine($"Packed RGB: 0x{color:X6}");  // 0xFF803C

        // --- Real-world: FileAccess flags from System.IO ---
        var access = System.IO.FileAccess.Read | System.IO.FileAccess.Write;
        Console.WriteLine($"FileAccess: {access}");  // "ReadWrite"
    }
}
```

### JavaScript

```javascript
// --- Basic OR ---
const a = 0b11001010;  // 202
const b = 0b10110110;  // 182
const result = a | b;  // 0b11111110 = 254

console.log(`a     = ${a.toString(2).padStart(8, '0')} (${a})`);
console.log(`b     = ${b.toString(2).padStart(8, '0')} (${b})`);
console.log(`a | b = ${result.toString(2).padStart(8, '0')} (${result})`);

// --- Setting a specific bit ---
let flags = 0b00001000;  // bit 3 set
const mask = 0b00000100; // target bit 2
flags |= mask;
console.log(`After setting bit 2: ${flags.toString(2).padStart(8, '0')}`);

// --- Combining flags (permission system) ---
const PERM = Object.freeze({
    NONE:    0,        // 0b0000
    READ:    1 << 0,   // 0b0001
    WRITE:   1 << 1,   // 0b0010
    EXECUTE: 1 << 2,   // 0b0100
    DELETE:  1 << 3,   // 0b1000
});

let userPerms = PERM.READ | PERM.WRITE | PERM.EXECUTE;
console.log(`User perms: ${userPerms.toString(2).padStart(4, '0')} (${userPerms})`);

// Check if a permission is set (uses AND)
if (userPerms & PERM.WRITE) {
    console.log("User has WRITE permission");
}

// --- Packing RGB ---
const r = 0xFF, g = 0x80, bVal = 0x3C;
const color = (r << 16) | (g << 8) | bVal;
console.log(`Packed RGB: 0x${color.toString(16).toUpperCase()}`);  // 0xFF803C

// --- BigInt OR for values beyond 32 bits ---
const big1 = 0xFFFFFFFFFFn;  // 40 bits, all 1s
const big2 = 0x1000000000n;
const bigResult = big1 | big2;
console.log(`BigInt OR: 0x${bigResult.toString(16)}`);
```

---

## ASCII Diagram -- OR Operation on 8-Bit Values

```
  OR Operation:  "If EITHER bit is 1, the result is 1"

  +---------+     +---------+
  | A: 0x4A |     | B: 0x35 |
  +---------+     +---------+
       |               |
       v               v
  0 1 0 0 1 0 1 0     (A = 0x4A = 74)
  0 0 1 1 0 1 0 1     (B = 0x35 = 53)
  ─ ─ ─ ─ ─ ─ ─ ─
  | | | | | | | |     Apply OR to each column:
  v v v v v v v v
  0 1 1 1 1 1 1 1     Result = 0x7F = 127
  +---------+
  | R: 0x7F |
  +---------+

  Bits that were 0 in A got "filled in" by the 1s from B.
  Bits that were 1 in A stayed 1 regardless of B.

  Think of OR as a UNION of set bits from both operands.
```

---

## Properties of the OR Operator

Understanding these algebraic properties helps when simplifying or reasoning about bitwise expressions:

| Property        | Expression                | Meaning                                    |
|-----------------|---------------------------|--------------------------------------------|
| Commutative     | `a \| b == b \| a`       | Order does not matter                      |
| Associative     | `(a \| b) \| c == a \| (b \| c)` | Grouping does not matter           |
| Idempotent      | `a \| a == a`             | ORing a value with itself yields itself    |
| Identity        | `a \| 0 == a`             | ORing with zero changes nothing            |
| Annihilator     | `a \| ~0 == ~0`           | ORing with all-1s yields all-1s            |
| Absorption      | `a \| (a & b) == a`       | Redundant AND term absorbed                |
| De Morgan's     | `~(a \| b) == ~a & ~b`   | Complement of OR equals AND of complements |

```ad-note
title: De Morgan's Law
De Morgan's law is essential when you need to invert conditions. Instead of negating a combined OR expression, you can negate each operand individually and switch to [[AND Operator]]. This is used constantly in both hardware design and software flag checks. See also [[NOT Operator]].
```

---

## OR vs Other Bitwise Operators

| Operation | Symbol | Result is `1` when...                         | Use case                   |
|-----------|--------|-----------------------------------------------|----------------------------|
| OR        | `\|`   | **At least one** input is `1`                 | Setting bits, combining    |
| AND       | `&`    | **Both** inputs are `1`                       | Checking bits, masking     |
| XOR       | `^`    | **Exactly one** input is `1` (inputs differ)  | Toggling bits, swapping    |
| NOT       | `~`    | Input is `0` (inverts)                        | Inverting masks            |

See [[AND Operator]], [[XOR Operator]], and [[NOT Operator]] for full details on each.

---

## Common Pitfalls

```ad-warning
title: Bitwise OR vs Logical OR
Do not confuse `|` (bitwise OR) with `||` (logical OR). The logical OR short-circuits and returns a boolean, while the bitwise OR evaluates both operands and operates on each bit. In C# this distinction is strict -- using `|` on booleans evaluates both sides (no short-circuit). In JavaScript, `|` coerces to 32-bit integers, so `true | false` gives `1`, not `true`.
```

```ad-warning
title: Forgetting Operator Precedence
Bitwise operators have **lower precedence** than comparison operators in most languages. This means `if (flags | mask == expected)` is parsed as `if (flags | (mask == expected))`, which is almost certainly not what you want. Always use parentheses: `if ((flags | mask) == expected)`.
```

```ad-warning
title: Using OR When You Need XOR
OR **cannot toggle** a bit -- it can only set it to `1`. If the bit is already `1` and you OR it again, it stays `1`. If you need to flip a bit (toggle between `0` and `1`), use [[XOR Operator]] instead.
```

---

## Related Topics

- [[AND Operator]] -- checking and masking bits
- [[XOR Operator]] -- toggling bits and finding differences
- [[NOT Operator]] -- inverting all bits
- [[Flag Enums and Bit Flags]] -- designing flag-based enumerations
- [[Check Set and Clear Bits]] -- the complete set/clear/toggle/check toolkit
- [[Permissions and Access Flags]] -- real-world flag systems
- [[Binary Number System]] -- foundations of binary representation
- [[Creating Bit Masks]] -- constructing masks for bitwise operations
- [[Left Shift]] -- positioning bits before ORing them together
- [[Combining and Extracting with Masks]] -- packing and unpacking bit fields
