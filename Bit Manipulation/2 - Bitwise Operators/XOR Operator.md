---
title: "XOR Operator"
date: 2026-08-19
tags:
  - bit-manipulation
  - operator
  - bitwise-xor
aliases:
  - Exclusive OR
  - XOR
  - Bitwise XOR
status: complete
---

## XOR Operator (Exclusive OR)

```ad-note
title: Overview
The XOR (Exclusive OR) operator compares two bits and returns `1` only when the bits are **different**. It is one of the most versatile [[AND Operator|bitwise operators]], with applications ranging from [[Toggle Bits|toggling bits]] and [[Swap Without Temp Variable|swapping variables]] to cryptography and error detection. XOR's unique mathematical properties -- self-inverse, identity, commutativity, and associativity -- make it indispensable in low-level programming, algorithm design, and hardware logic.
```

---

## Table of Contents

- [[#Truth Table]]
- [[#Bit-by-Bit Operation]]
- [[#Key Properties of XOR]]
  - [[#Self-Inverse Property]]
  - [[#Identity Property]]
  - [[#Commutative Property]]
  - [[#Associative Property]]
- [[#Syntax Across Languages]]
- [[#Practical Uses]]
  - [[#Toggling Bits]]
  - [[#Swapping Without a Temporary Variable]]
  - [[#Finding the Lone Non-Duplicate Element]]
  - [[#Simple Encryption and Checksums]]
- [[#Full Code Examples]]
  - [[#C++ Example]]
  - [[#C-Sharp Example]]
  - [[#JavaScript Example]]
- [[#XOR Operation Diagram]]
- [[#Common Pitfalls and Misconceptions]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## Truth Table

The XOR operator outputs `1` **only** when the two input bits differ. If they are the same (both `0` or both `1`), the result is `0`.

| A | B | A ^ B |
|---|---|:-----:|
| 0 | 0 |   0   |
| 0 | 1 |   1   |
| 1 | 0 |   1   |
| 1 | 1 |   0   |

Compare this with the [[AND Operator]] (both must be `1`) and the [[OR Operator]] (at least one must be `1`). XOR is the **only** basic bitwise operator that returns `0` when both inputs are `1` -- that exclusivity is what gives it the name "Exclusive OR."

```ad-summary
title: Section Summary
- XOR returns `1` when inputs differ, `0` when they match.
- It is "exclusive" because it excludes the case where both bits are `1`.
- Contrast with [[AND Operator]] (both `1`) and [[OR Operator]] (at least one `1`).
```

---

## Bit-by-Bit Operation

XOR operates on each bit position independently. Here is a visual walkthrough of `0b11001010 ^ 0b10101100`:

```
    Decimal:  202         ^        172         =         102
    
    Bit Position:   7   6   5   4   3   2   1   0
                  +---+---+---+---+---+---+---+---+
    A (202):      | 1 | 1 | 0 | 0 | 1 | 0 | 1 | 0 |
                  +---+---+---+---+---+---+---+---+
    B (172):      | 1 | 0 | 1 | 0 | 1 | 1 | 0 | 0 |
                  +---+---+---+---+---+---+---+---+
                    |   |   |   |   |   |   |   |
                   same diff diff same same diff diff same
                    |   |   |   |   |   |   |   |
                  +---+---+---+---+---+---+---+---+
    A ^ B (102):  | 0 | 1 | 1 | 0 | 0 | 1 | 1 | 0 |
                  +---+---+---+---+---+---+---+---+
```

Each result bit is `1` where A and B **differ**, and `0` where they **match**. This is the fundamental mechanism behind every XOR application: it is a **difference detector** at the bit level.

```ad-summary
title: Section Summary
- XOR compares each bit position independently.
- Result bit is `1` where the operand bits differ.
- XOR is essentially a per-bit difference detector.
```

---

## Key Properties of XOR

XOR has four algebraic properties that make it uniquely powerful among bitwise operators. Understanding these properties is essential for recognizing where XOR can be applied.

### Self-Inverse Property

```
a ^ a = 0
```

Any value XORed with itself produces zero. Every bit cancels out because identical bits always yield `0`. This is the foundation of the [[Find the Lone Non-Duplicate]] algorithm and the [[Swap Without Temp Variable]] trick.

```
    0b10110011
  ^ 0b10110011
  ────────────
    0b00000000    (all bits match, so all results are 0)
```

### Identity Property

```
a ^ 0 = a
```

XORing any value with zero leaves it unchanged. Zero is the **identity element** of XOR. No bits flip because `0` differs from nothing.

```
    0b10110011
  ^ 0b00000000
  ────────────
    0b10110011    (original value preserved)
```

### Commutative Property

```
a ^ b = b ^ a
```

The order of operands does not matter. This means you can rearrange XOR chains freely without changing the result.

### Associative Property

```
(a ^ b) ^ c = a ^ (b ^ c)
```

Grouping does not matter. Combined with commutativity, this means you can XOR a sequence of values in **any order** and get the same result. This is why XORing all elements in an array works to [[Find the Lone Non-Duplicate]] -- pairs cancel regardless of position.

```ad-tip
title: Why These Properties Matter Together
Commutativity + Associativity + Self-Inverse together mean: in any collection of values, **every value that appears an even number of times cancels to zero**, leaving only values that appear an odd number of times. This single insight powers numerous algorithms.
```

```ad-summary
title: Section Summary
- `a ^ a = 0` -- self-inverse, any value cancels itself.
- `a ^ 0 = a` -- identity, XOR with zero preserves the value.
- `a ^ b = b ^ a` -- commutative, order does not matter.
- `(a ^ b) ^ c = a ^ (b ^ c)` -- associative, grouping does not matter.
- Combined, these properties allow pair-cancellation in any order.
```

---

## Syntax Across Languages

The XOR operator uses the caret symbol `^` in C++, C#, and JavaScript. Despite shared syntax, there are language-specific nuances.

| Language   | Operator | Types                            | Notes                                              |
|------------|:--------:|----------------------------------|----------------------------------------------------|
| C++        |   `^`    | Integer types, `bool`            | Works on all integral types; no overflow concern    |
| C#         |   `^`    | `int`, `uint`, `long`, `bool`    | Also works as logical XOR on `bool` operands       |
| JavaScript |   `^`    | Numbers (coerced to 32-bit int)  | Operands converted to signed 32-bit before XOR     |

```ad-warning
title: JavaScript 32-Bit Truncation
In JavaScript, the `^` operator converts both operands to **signed 32-bit integers** before performing XOR. This means values larger than `2^31 - 1` or using more than 32 bits will be truncated. Be careful when working with large numbers or BigInt (use `BigInt` XOR via separate handling).
```

```ad-note
title: C# Boolean XOR
In C#, `^` on two `bool` operands performs a **logical XOR** -- it returns `true` when exactly one operand is `true`. This is different from `!=` in subtle ways when combined with nullable booleans.
```

```ad-summary
title: Section Summary
- All three languages use `^` for XOR.
- C++ and C# apply XOR to all integer types directly.
- C# also supports `^` as logical XOR on `bool`.
- JavaScript coerces operands to 32-bit signed integers first.
```

---

## Practical Uses

### Toggling Bits

XOR is the standard way to [[Toggle Bits|toggle (flip) specific bits]] in a value. XORing a bit with `1` flips it; XORing with `0` leaves it unchanged.

```
value  = 0b11010010
mask   = 0b00001111   (toggle the lower 4 bits)
result = 0b11011101   (lower 4 bits flipped, upper 4 unchanged)
```

This is how you flip flags, invert selections, or alternate states without branching. See also [[Check Set and Clear Bits]] and [[Creating Bit Masks]] for the complementary operations of reading and clearing bits.

### Swapping Without a Temporary Variable

The self-inverse property enables the classic [[Swap Without Temp Variable]] trick:

```
a = a ^ b    // a now holds combined difference
b = a ^ b    // b becomes original a  (combined ^ original b = original a)
a = a ^ b    // a becomes original b  (combined ^ original a = original b)
```

```ad-warning
title: Same-Variable Trap
If `a` and `b` refer to the **same memory location** (e.g., `swap(arr, i, i)` where both indices are equal), XOR swap zeroes out the value because `a ^ a = 0`. Always guard against this case. In modern code, a temporary variable or `std::swap` is preferred for clarity and safety.
```

### Finding the Lone Non-Duplicate Element

Given an array where every element appears exactly twice except one, XOR all elements together. Pairs cancel via `a ^ a = 0`, and the lone element survives via `result ^ 0 = result`. This runs in **O(n)** time and **O(1)** space -- see [[Find the Lone Non-Duplicate]] for the full algorithm and extensions (e.g., finding two unique elements).

### Simple Encryption and Checksums

XOR is used in:

- **XOR cipher**: `ciphertext = plaintext ^ key`. Applying the same key again decrypts: `plaintext = ciphertext ^ key`. This is the basis of the one-time pad (provably unbreakable when the key is truly random, as long as the message, and never reused).
- **Checksums and parity**: XOR all bytes of a message to produce a simple error-detection checksum. If any single bit flips, the checksum changes.
- **RAID 5 parity**: Disk striping uses XOR across drives so that any single drive failure can be reconstructed from the remaining drives.
- **CRC and hash mixing**: Many hash functions use XOR to combine partial hash values.

```ad-warning
title: XOR Cipher Weakness
A simple XOR cipher with a short, repeating key is trivially breakable via frequency analysis. XOR encryption is only secure with a one-time pad -- a key that is truly random, at least as long as the plaintext, and used exactly once.
```

```ad-summary
title: Section Summary
- [[Toggle Bits]]: XOR with a mask to flip selected bits.
- [[Swap Without Temp Variable]]: three XOR operations swap two values in-place.
- [[Find the Lone Non-Duplicate]]: XOR all array elements to isolate the unique one.
- Encryption: XOR cipher is reversible; secure only as a one-time pad.
- Checksums: XOR bytes together for simple error detection.
```

---

## Full Code Examples

### C++ Example

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    // --- Basic XOR ---
    unsigned char a = 0b11001010;  // 202
    unsigned char b = 0b10101100;  // 172
    unsigned char result = a ^ b;  // 0b01100110 = 102
    cout << "202 ^ 172 = " << (int)result << endl;

    // --- Toggle Bits ---
    unsigned char flags = 0b11010010;
    unsigned char mask  = 0b00001111;
    flags ^= mask;  // lower 4 bits toggled
    cout << "Toggled flags: " << (int)flags << endl;

    // --- Swap Without Temp ---
    int x = 42, y = 99;
    if (&x != &y) {  // guard against same-address
        x ^= y;
        y ^= x;
        x ^= y;
    }
    cout << "After swap: x=" << x << ", y=" << y << endl;

    // --- Find Lone Non-Duplicate ---
    vector<int> nums = {4, 1, 2, 1, 2};
    int lone = 0;
    for (int n : nums) {
        lone ^= n;
    }
    cout << "Lone element: " << lone << endl;

    // --- Simple XOR Cipher ---
    string message = "Hello";
    char key = 0x5A;
    string encrypted = message;
    for (char& c : encrypted) {
        c ^= key;
    }
    // Decrypt by applying the same key again
    string decrypted = encrypted;
    for (char& c : decrypted) {
        c ^= key;
    }
    cout << "Decrypted: " << decrypted << endl;

    return 0;
}
```

### C-Sharp Example

```csharp
using System;
using System.Linq;

class XorDemo
{
    static void Main()
    {
        // --- Basic XOR ---
        int a = 0b_1100_1010;  // 202
        int b = 0b_1010_1100;  // 172
        int result = a ^ b;    // 0b_0110_0110 = 102
        Console.WriteLine($"202 ^ 172 = {result}");

        // --- Toggle Bits ---
        int flags = 0b_1101_0010;
        int mask  = 0b_0000_1111;
        flags ^= mask;
        Console.WriteLine($"Toggled flags: {Convert.ToString(flags, 2).PadLeft(8, '0')}");

        // --- Swap Without Temp ---
        int x = 42, y = 99;
        x ^= y;
        y ^= x;
        x ^= y;
        Console.WriteLine($"After swap: x={x}, y={y}");

        // --- Find Lone Non-Duplicate ---
        int[] nums = { 4, 1, 2, 1, 2 };
        int lone = nums.Aggregate(0, (acc, n) => acc ^ n);
        Console.WriteLine($"Lone element: {lone}");

        // --- Boolean XOR (C# specific) ---
        bool p = true, q = false;
        bool logicalXor = p ^ q;  // true (exactly one is true)
        Console.WriteLine($"true ^ false = {logicalXor}");

        // --- Simple XOR Cipher ---
        string message = "Hello";
        byte key = 0x5A;
        char[] encrypted = message.Select(c => (char)(c ^ key)).ToArray();
        char[] decrypted = encrypted.Select(c => (char)(c ^ key)).ToArray();
        Console.WriteLine($"Decrypted: {new string(decrypted)}");
    }
}
```

### JavaScript Example

```javascript
// --- Basic XOR ---
const a = 0b11001010;  // 202
const b = 0b10101100;  // 172
const result = a ^ b;  // 0b01100110 = 102
console.log(`202 ^ 172 = ${result}`);

// --- Toggle Bits ---
let flags = 0b11010010;
const mask = 0b00001111;
flags ^= mask;
console.log(`Toggled flags: ${flags.toString(2).padStart(8, '0')}`);

// --- Swap Without Temp ---
let x = 42, y = 99;
x ^= y;
y ^= x;
x ^= y;
console.log(`After swap: x=${x}, y=${y}`);

// --- Find Lone Non-Duplicate ---
const nums = [4, 1, 2, 1, 2];
const lone = nums.reduce((acc, n) => acc ^ n, 0);
console.log(`Lone element: ${lone}`);

// --- Simple XOR Cipher ---
const message = "Hello";
const key = 0x5A;
const encrypted = [...message].map(c => String.fromCharCode(c.charCodeAt(0) ^ key)).join('');
const decrypted = [...encrypted].map(c => String.fromCharCode(c.charCodeAt(0) ^ key)).join('');
console.log(`Decrypted: ${decrypted}`);

// --- 32-Bit Truncation Gotcha ---
const big = 2 ** 33;        // 8589934592 (beyond 32 bits)
console.log(big ^ 0);       // 0 -- the upper bits are lost!
console.log(big === 0);     // false -- the number itself is fine
// Use BigInt for values beyond 32 bits:
console.log(BigInt(2 ** 33) ^ 0n);  // 8589934592n (correct)
```

```ad-summary
title: Section Summary
- All three languages use `^` for XOR with nearly identical syntax.
- C++ requires casts for printing `unsigned char` as a number.
- C# supports `^` on `bool` for logical XOR and offers `Aggregate` for reducing arrays.
- JavaScript truncates to 32-bit signed integers; use `BigInt` for larger values.
- Each example demonstrates: basic XOR, toggling, swapping, lone-element finding, and XOR cipher.
```

---

## XOR Operation Diagram

Below is a visual representation of how XOR processes two 8-bit values through each bit position:

```
  INPUT A:    1   1   0   0   1   0   1   0       (0xCA = 202)
              |   |   |   |   |   |   |   |
             [XOR][XOR][XOR][XOR][XOR][XOR][XOR][XOR]   <-- per-bit XOR gates
              |   |   |   |   |   |   |   |
  INPUT B:    1   0   1   0   1   1   0   0       (0xAC = 172)
              |   |   |   |   |   |   |   |
              v   v   v   v   v   v   v   v
  OUTPUT:     0   1   1   0   0   1   1   0       (0x66 = 102)
              
  RULE:      same diff diff same same diff diff same
              =0   =1   =1   =0   =0   =1   =1   =0
```

XOR gate symbol for reference (standard logic gate):

```
         A ──────\
                  )>──── A XOR B
         B ──────/
         
    "Output is HIGH when inputs DIFFER"
```

---

## Common Pitfalls and Misconceptions

```ad-warning
title: Common Misconception -- XOR as "Addition Without Carry"
XOR is sometimes described as "binary addition without carry." While this is true for single bits (`0+1=1`, `1+1=0` in XOR), it can be misleading. XOR does not propagate carries, so `3 ^ 1` is `2`, not `4`. Full addition requires both XOR (for the sum bit) and AND (for the carry bit) -- this is how half-adders and full-adders work in hardware.
```

```ad-warning
title: Common Misconception -- XOR Swap is "Better"
The XOR swap trick is a classic interview puzzle, but in practice it is **not** superior to using a temporary variable. Modern compilers optimize a temp-based swap into register operations that are as fast or faster. XOR swap is harder to read, fails when both variables alias the same location, and can confuse branch predictors. Use `std::swap()` in C++, tuple unpacking in Python, or destructuring in JavaScript.
```

```ad-warning
title: Common Misconception -- XOR Encryption is Secure
XOR with a fixed or short repeating key is **not** secure encryption. It is vulnerable to known-plaintext attacks and frequency analysis. Only a one-time pad (key as long as the message, truly random, never reused) provides information-theoretic security. For real encryption, use established algorithms like AES.
```

```ad-tip
title: XOR for Detecting Changes
XOR is often used in diff-style operations: `old_state ^ new_state` produces a value where every `1` bit indicates a bit that changed. This is useful for tracking which flags were toggled, which pixels changed in a frame, or which configuration bits were modified.
```

```ad-summary
title: Section Summary
- XOR is binary addition without carry -- useful analogy but can mislead for multi-bit values.
- XOR swap is a clever trick but not practical; prefer language-standard swap utilities.
- XOR encryption requires a one-time pad to be secure.
- XOR excels as a change-detection tool: `old ^ new` reveals which bits flipped.
```

---

## Comprehensive Summary

```ad-tip
title: Complete Summary
The **XOR (Exclusive OR)** operator returns `1` when two bits differ and `0` when they match, making it a fundamental **difference detector** in the [[Binary Number System]].

**Core Properties:**
- **Self-inverse** (`a ^ a = 0`): any value cancels itself, enabling pair-elimination algorithms.
- **Identity** (`a ^ 0 = a`): XOR with zero is a no-op.
- **Commutative** and **Associative**: order and grouping do not matter, so XOR chains can be freely rearranged.

**Practical Applications:**
- **[[Toggle Bits]]**: XOR with a [[Creating Bit Masks|bitmask]] to flip specific bits without affecting others.
- **[[Swap Without Temp Variable]]**: three XOR operations swap two values in-place (but beware the same-variable trap).
- **[[Find the Lone Non-Duplicate]]**: XOR all elements to isolate the unique value in O(n) time, O(1) space.
- **Encryption**: the XOR cipher is the basis of the one-time pad; insecure with short or repeating keys.
- **Checksums and parity**: XOR all bytes for simple single-bit error detection (used in RAID, serial protocols).
- **Change detection**: `old ^ new` reveals exactly which bits changed between two states.

**Language Notes:**
- C++, C#, and JavaScript all use `^` for XOR.
- C# extends `^` to work as logical XOR on `bool` operands.
- JavaScript coerces operands to 32-bit signed integers, truncating values beyond that range.

XOR's combination of reversibility, order-independence, and per-bit difference detection makes it one of the most versatile operators in a programmer's toolkit. Its properties appear throughout algorithms, cryptography, error correction, hardware design, and competitive programming.
```

---

## Related Topics

- [[AND Operator]]
- [[OR Operator]]
- [[NOT Operator]]
- [[Toggle Bits]]
- [[Swap Without Temp Variable]]
- [[Find the Lone Non-Duplicate]]
- [[Binary Number System]]
- [[Check Set and Clear Bits]]
- [[Creating Bit Masks]]
