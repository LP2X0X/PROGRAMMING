---
tags:
  - bit-manipulation
  - javascript
  - language-specific
---

## Overview

JavaScript's relationship with bit manipulation is unique and full of surprises. All JavaScript numbers are IEEE 754 double-precision floating-point values, but **every bitwise operator converts its operands to 32-bit signed integers** before operating. Understanding this conversion is the single most important thing for doing bit manipulation in JavaScript correctly.

---

## The Fundamental Gotcha: 64-bit Floats, 32-bit Bitwise

```ad-warning
title: Critical -- All bitwise operators work on 32-bit signed integers
JavaScript has only one number type: IEEE 754 double-precision floating-point (64 bits). But ALL bitwise operators (`&`, `|`, `^`, `~`, `<<`, `>>`) internally:
1. Convert each operand to a **32-bit signed integer** (two's complement)
2. Perform the bitwise operation
3. Convert the result back to a 64-bit double

The exception is `>>>` (unsigned right shift), which produces an **unsigned** 32-bit result.

This means:
- Large numbers (> 2^31 - 1) are silently **truncated** to 32 bits
- Fractional parts are **dropped** (truncated toward zero)
- Results are always in the range -2^31 to 2^31 - 1 (except `>>>`)
```

See [[Signed and Unsigned Integers]] and [[Twos Complement]] for the representations involved.

### Demonstrating the Conversion

```javascript
// Numbers are 64-bit doubles
let big = 2 ** 53 - 1;  // 9007199254740991 (Number.MAX_SAFE_INTEGER)
console.log(big);        // 9007199254740991

// Bitwise OR with 0 forces 32-bit conversion
console.log(big | 0);   // -1 (truncated to 32 bits!)

// Fractional parts are dropped
console.log(3.9 | 0);   // 3 (not 4 -- truncates, doesn't round)
console.log(-3.9 | 0);  // -3

// Values > 2^31 - 1 get mangled
let largePositive = 0xFFFFFFFF;  // 4294967295 (fits in 64-bit double)
console.log(largePositive | 0);   // -1 (interpreted as signed 32-bit)
```

### How the Conversion Works (ToInt32)

The internal `ToInt32` conversion follows these steps:

1. Convert to Number (call `ToNumber`)
2. If NaN, +0, -0, +Infinity, or -Infinity, return +0
3. Truncate toward zero (drop fractional part)
4. Reduce modulo 2^32 to get a value in `[0, 2^32)`
5. If the result >= 2^31, subtract 2^32 (to get the signed representation)

```javascript
// Step by step for a large number:
let n = 4294967296;  // 2^32
// Step 3: truncate -> 4294967296
// Step 4: modulo 2^32 -> 0
// Step 5: 0 < 2^31, so result is 0
console.log(n | 0);  // 0

let m = 4294967297;  // 2^32 + 1
// modulo 2^32 -> 1
console.log(m | 0);  // 1

let p = 2147483648;  // 2^31
// modulo 2^32 -> 2147483648
// 2147483648 >= 2^31, so subtract 2^32 -> -2147483648
console.log(p | 0);  // -2147483648
```

```ad-warning
title: Silent data loss
The 32-bit conversion happens silently with no warning or error. If you apply bitwise operators to a number larger than 2^31 - 1, JavaScript will quietly give you the wrong answer. There is no runtime check.
```

---

## Bitwise Operators

JavaScript has seven bitwise operators. Six produce signed 32-bit results; one (`>>>`) produces an unsigned 32-bit result.

| Operator | Name                    | Result Type          | Link                         |
|----------|-------------------------|----------------------|------------------------------|
| `&`      | Bitwise AND             | Signed 32-bit        | [[AND Operator]]             |
| `\|`     | Bitwise OR              | Signed 32-bit        | [[OR Operator]]              |
| `^`      | Bitwise XOR             | Signed 32-bit        | [[XOR Operator]]             |
| `~`      | Bitwise NOT             | Signed 32-bit        | [[NOT Operator]]             |
| `<<`     | Left shift              | Signed 32-bit        | [[Left Shift]]               |
| `>>`     | Sign-propagating right shift | Signed 32-bit   | [[Right Shift]]              |
| `>>>`    | Zero-fill right shift   | **Unsigned 32-bit**  | [[Unsigned Right Shift]]     |

### Compound Assignment Operators

All binary bitwise operators have compound assignment forms:

```javascript
let flags = 0b1010;
flags &= 0b1100;   // flags = flags & 0b1100  --> 0b1000
flags |= 0b0011;   // flags = flags | 0b0011  --> 0b1011
flags ^= 0b0101;   // flags = flags ^ 0b0101  --> 0b1110
flags <<= 2;        // flags = flags << 2      --> 0b111000
flags >>= 1;        // flags = flags >> 1      --> 0b11100
flags >>>= 0;       // flags = flags >>> 0     --> same value but as unsigned
```

---

## Shift Operators in Detail

### Left Shift (`<<`)

Shifts bits left. Vacated positions on the right are filled with zeros. The result is a signed 32-bit integer.

```javascript
let value = 1;
console.log(value << 0);   // 1
console.log(value << 1);   // 2
console.log(value << 10);  // 1024
console.log(value << 31);  // -2147483648 (MSB set = negative in signed 32-bit)

// Only the low 5 bits of the shift amount are used (shift mod 32)
console.log(1 << 32);  // 1 (same as << 0, NOT 0!)
console.log(1 << 33);  // 2 (same as << 1)
```

```ad-warning
title: Shift amount is modulo 32
JavaScript uses only the low 5 bits of the shift amount, effectively computing `shift % 32`. So `x << 32` is the same as `x << 0`, not zero! This is different from some other languages.
```

### Sign-Propagating Right Shift (`>>`)

Shifts bits right. The sign bit (MSB) is copied into the vacated positions on the left. This preserves the sign of the number (arithmetic shift). See [[Right Shift]].

```javascript
let positive = 100;           // 0b...01100100
console.log(positive >> 2);   // 25 (0b...00011001)

let negative = -100;          // 0b...10011100 (two's complement)
console.log(negative >> 2);   // -25 (sign bit preserved)
console.log(negative >> 31);  // -1 (all bits become 1 from sign propagation)
```

### Zero-Fill Right Shift (`>>>`)

Shifts bits right. The vacated positions on the left are always filled with zeros, regardless of the sign bit. This is the **only** bitwise operator in JavaScript that produces an unsigned 32-bit result. See [[Unsigned Right Shift]].

```javascript
let positive = 100;
console.log(positive >>> 2);   // 25 (same as >> for positive numbers)

let negative = -100;
console.log(negative >> 2);    // -25 (sign-preserving)
console.log(negative >>> 2);   // 1073741799 (zero-filled, unsigned result)

// The classic trick: convert to unsigned 32-bit
console.log(-1 >>> 0);         // 4294967295 (0xFFFFFFFF as unsigned)
console.log(-1 >> 0);          // -1 (still signed)
```

```ad-tip
title: >>> 0 as unsigned conversion
`n >>> 0` is the standard JavaScript idiom for converting a number to an unsigned 32-bit integer. It performs the 32-bit conversion and then a zero-fill shift by 0 positions, returning the unsigned interpretation. This is commonly used when you need to treat a value as unsigned.
```

---

## Binary and Hexadecimal Literals

### Binary Literals (`0b` prefix, ES2015+)

```javascript
let flags = 0b10101100;     // 172
let mask  = 0b00001111;     // 15 (lower nibble mask)
let bit5  = 0b00100000;     // 32 (bit 5 set)
```

### Hexadecimal Literals (`0x` prefix)

```javascript
let color = 0xFF00FF;       // 16711935 (magenta RGB)
let mask  = 0xF0;           // 240 (upper nibble mask)
let addr  = 0xDEADBEEF;    // 3735928559
```

### Octal Literals (`0o` prefix, ES2015+)

```javascript
let filePerms = 0o755;      // 493 (rwxr-xr-x in Unix)
let mask = 0o777;           // 511
```

```ad-note
title: No digit separators in JavaScript
Unlike C++ (which uses `'`) and C# (which uses `_`), JavaScript does not support digit separators in numeric literals. The TC39 proposal for numeric separators (`1_000_000`) was added in ES2021, but only for decimal, hex, octal, and binary -- not universally available in all environments.
```

Actually, numeric separators were added in ES2021:

```javascript
// ES2021+ numeric separators (underscore)
let billion = 1_000_000_000;
let hex     = 0xFF_FF_FF_FF;
let binary  = 0b1010_0101_1100_0011;
let octal   = 0o777_777;
```

```ad-tip
title: Check your target environment
Numeric separators (`_`) in JavaScript require ES2021 or later. If targeting older environments, you may need a transpiler (Babel) or cannot use them.
```

---

## The >>> Operator: JavaScript's Unique Feature

The zero-fill right shift operator (`>>>`) is unique to JavaScript among the three languages covered in this folder. It is the **only** JavaScript bitwise operator that produces an unsigned 32-bit result.

### Key Behaviors

```javascript
// Positive numbers: >>> and >> behave identically
console.log(100 >>> 2);   // 25
console.log(100 >> 2);    // 25

// Negative numbers: very different results
console.log(-1 >> 0);     // -1
console.log(-1 >>> 0);    // 4294967295  (0xFFFFFFFF unsigned)

console.log(-100 >> 4);   // -7
console.log(-100 >>> 4);  // 268435449

// Common idiom: convert to unsigned 32-bit integer
function toUint32(n) {
    return n >>> 0;
}

console.log(toUint32(-1));        // 4294967295
console.log(toUint32(3.7));       // 3
console.log(toUint32(-3.7));      // 4294967293
console.log(toUint32(2**32));     // 0 (wraps)
console.log(toUint32(2**32 + 5)); // 5
```

### Why >>> Matters

```javascript
// In other languages you can declare uint32. In JS you cannot.
// >>> 0 is the closest equivalent.

// Example: reading unsigned values from binary data
let buffer = new ArrayBuffer(4);
let view = new DataView(buffer);
view.setUint32(0, 0xFFFFFFFF);

// DataView gives you the correct unsigned value
let unsigned = view.getUint32(0);  // 4294967295

// But if you get a signed int and need unsigned:
let signed = view.getInt32(0);     // -1
let asUnsigned = signed >>> 0;     // 4294967295
```

---

## Typed Arrays and DataView

For serious binary data manipulation, JavaScript provides typed arrays and DataView. These are essential when working with binary protocols, file formats, or WebAssembly.

### ArrayBuffer: The Raw Binary Buffer

```javascript
// Create a 16-byte buffer (all zeros)
let buffer = new ArrayBuffer(16);

// Check size
console.log(buffer.byteLength);  // 16

// Cannot read/write directly -- need a view
```

### Typed Arrays: Fixed-Type Views

Typed arrays provide typed access to an `ArrayBuffer`:

| Typed Array       | Bytes per Element | Description                   | Value Range              |
|-------------------|-------------------|-------------------------------|--------------------------|
| `Int8Array`       | 1                 | 8-bit signed integer          | -128 to 127              |
| `Uint8Array`      | 1                 | 8-bit unsigned integer        | 0 to 255                 |
| `Uint8ClampedArray` | 1              | 8-bit unsigned, clamped       | 0 to 255 (clamped)       |
| `Int16Array`      | 2                 | 16-bit signed integer         | -32768 to 32767          |
| `Uint16Array`     | 2                 | 16-bit unsigned integer       | 0 to 65535               |
| `Int32Array`      | 4                 | 32-bit signed integer         | -2^31 to 2^31-1          |
| `Uint32Array`     | 4                 | 32-bit unsigned integer       | 0 to 2^32-1              |
| `BigInt64Array`   | 8                 | 64-bit signed BigInt          | -2^63 to 2^63-1          |
| `BigUint64Array`  | 8                 | 64-bit unsigned BigInt        | 0 to 2^64-1              |
| `Float32Array`    | 4                 | 32-bit IEEE 754 float         |                           |
| `Float64Array`    | 8                 | 64-bit IEEE 754 float         |                           |

```javascript
// Create from length
let bytes = new Uint8Array(8);           // 8 bytes, all zeros
let ints  = new Int32Array(4);           // 4 int32s = 16 bytes

// Create from array
let data = new Uint8Array([0xFF, 0x00, 0xAA, 0x55]);

// Create a view over an existing buffer
let buffer = new ArrayBuffer(8);
let view8  = new Uint8Array(buffer);        // View all 8 bytes as uint8
let view32 = new Uint32Array(buffer);       // View as 2 uint32s
let partial = new Uint16Array(buffer, 2, 2); // 2 uint16s starting at byte offset 2

// Typed arrays share the underlying buffer!
view8[0] = 0xFF;
view8[1] = 0x00;
view8[2] = 0x00;
view8[3] = 0x01;
console.log(view32[0]);  // Depends on endianness! (little-endian: 0x010000FF)
```

```ad-warning
title: Typed arrays use native endianness
When you interpret bytes through typed arrays (e.g., reading a `Uint32Array` over a `Uint8Array` buffer), the byte order depends on the system's native endianness. Most systems are little-endian, but this is not guaranteed. For explicit endianness control, use `DataView`. See [[Endianness]].
```

### DataView: Explicit Endianness Control

`DataView` provides methods for reading and writing specific types at specific byte offsets with explicit endianness. This is the correct tool for parsing binary protocols and file formats.

```javascript
let buffer = new ArrayBuffer(8);
let view = new DataView(buffer);

// Write a 32-bit unsigned int in big-endian (network byte order)
view.setUint32(0, 0x0A0B0C0D, false);  // false = big-endian
// buffer: [0x0A, 0x0B, 0x0C, 0x0D, 0x00, 0x00, 0x00, 0x00]

// Write a 32-bit unsigned int in little-endian
view.setUint32(4, 0x01020304, true);   // true = little-endian
// buffer: [0x0A, 0x0B, 0x0C, 0x0D, 0x04, 0x03, 0x02, 0x01]

// Read back with explicit endianness
let bigEndian = view.getUint32(0, false);     // 0x0A0B0C0D
let littleEndian = view.getUint32(0, true);   // 0x0D0C0B0A
```

### Available DataView Methods

| Read Method            | Write Method            | Size   |
|------------------------|------------------------|--------|
| `getInt8(offset)`      | `setInt8(offset, val)` | 1 byte |
| `getUint8(offset)`     | `setUint8(offset, val)`| 1 byte |
| `getInt16(o, le)`      | `setInt16(o, val, le)` | 2 bytes|
| `getUint16(o, le)`     | `setUint16(o, val, le)`| 2 bytes|
| `getInt32(o, le)`      | `setInt32(o, val, le)` | 4 bytes|
| `getUint32(o, le)`     | `setUint32(o, val, le)`| 4 bytes|
| `getBigInt64(o, le)`   | `setBigInt64(o, val, le)` | 8 bytes|
| `getBigUint64(o, le)`  | `setBigUint64(o, val, le)` | 8 bytes|
| `getFloat32(o, le)`    | `setFloat32(o, val, le)` | 4 bytes|
| `getFloat64(o, le)`    | `setFloat64(o, val, le)` | 8 bytes|

The `le` parameter is a boolean: `true` for little-endian, `false` for big-endian. When omitted, defaults to big-endian.

```ad-tip
title: DataView default is big-endian
DataView defaults to big-endian (network byte order) when the `littleEndian` parameter is omitted. This is the opposite of what most modern hardware uses (x86/ARM are little-endian). Always pass the endianness parameter explicitly to avoid confusion.
```

---

## BigInt for Arbitrary-Precision Bit Manipulation

`BigInt` (ES2020) provides arbitrary-precision integers. Bitwise operators work on `BigInt` values, but you cannot mix `BigInt` and `Number` in the same expression.

### Creating BigInts

```javascript
let a = 123n;                              // BigInt literal (append n)
let b = BigInt(456);                       // From number
let c = BigInt("0xFF");                    // From hex string
let d = BigInt("0b101010");               // From binary string
let huge = 123456789012345678901234567890n; // Arbitrary precision
```

### Bitwise Operators on BigInt

All bitwise operators work on `BigInt`, operating on the full precision:

```javascript
let a = 0b1010n;
let b = 0b1100n;

console.log(a & b);   // 8n   (0b1000) -- see [[AND Operator]]
console.log(a | b);   // 14n  (0b1110) -- see [[OR Operator]]
console.log(a ^ b);   // 6n   (0b0110) -- see [[XOR Operator]]
console.log(~a);       // -11n (bitwise NOT on arbitrary precision)
console.log(a << 2n);  // 40n  (0b101000) -- see [[Left Shift]]
console.log(a >> 1n);  // 5n   (0b0101) -- see [[Right Shift]]
```

```ad-warning
title: Cannot mix BigInt and Number
Mixing BigInt and Number in the same operation is a TypeError. You must convert explicitly.
```

```javascript
let big = 10n;
let num = 5;

// TypeError: Cannot mix BigInt and other types
// let result = big + num;
// let result = big & num;
// let result = big << num;

// Must convert:
let result1 = big + BigInt(num);   // 15n
let result2 = big & BigInt(num);   // 0n
let result3 = big << BigInt(num);  // 320n
```

### BigInt vs Number for Bitwise Work

```javascript
// Number: limited to 32-bit signed for bitwise ops
let numResult = 0xFFFFFFFF | 0;  // -1 (32-bit signed conversion)

// BigInt: full precision maintained
let bigResult = 0xFFFFFFFFn | 0n;  // 4294967295n (no truncation)

// 64-bit operations? Easy with BigInt
let val64 = 0xDEADBEEF_CAFEBABEn;
let mask64 = 0xFFFFFFFF_00000000n;
let upper = (val64 & mask64) >> 32n;  // 0xDEADBEEFn
console.log(upper.toString(16));      // "deadbeef"
```

```ad-tip
title: When to use BigInt for bit manipulation
Use BigInt when you need:
- More than 32 bits of precision in bitwise operations
- 64-bit integer operations (e.g., parsing 64-bit fields from binary data)
- Cryptographic operations requiring large integers
- Any bitwise work where 32-bit truncation would lose data

For standard 32-bit bit manipulation, regular Number with bitwise operators is simpler and faster.
```

---

## Common JavaScript Bit Tricks

### `~~x` -- Double NOT as Integer Truncation

The double bitwise NOT (`~~`) is a fast way to truncate a number to a 32-bit integer. It converts to 32-bit signed, applies NOT twice (returning to original bits), and gives back the truncated integer.

```javascript
console.log(~~3.7);      // 3
console.log(~~-3.7);     // -3
console.log(~~"42");     // 42 (string converted to number first)
console.log(~~"hello");  // 0 (NaN -> 0)
console.log(~~true);     // 1
console.log(~~null);     // 0
console.log(~~undefined);// 0
```

```ad-warning
title: ~~ fails for large numbers
`~~` truncates to 32 bits, so it gives wrong results for numbers outside the 32-bit signed range (-2^31 to 2^31 - 1). Use `Math.trunc()` for a safe alternative that works with all Number values.
```

```javascript
console.log(~~2147483648);     // -2147483648 (WRONG -- overflow)
console.log(Math.trunc(2147483648));  // 2147483648 (correct)
```

### `x | 0` -- Convert to 32-bit Signed Integer

```javascript
console.log(3.14 | 0);     // 3
console.log(-3.14 | 0);    // -3
console.log("42" | 0);     // 42
console.log(NaN | 0);      // 0
console.log(Infinity | 0); // 0
```

### `x >>> 0` -- Convert to Unsigned 32-bit Integer

```javascript
console.log(-1 >>> 0);        // 4294967295
console.log(3.14 >>> 0);      // 3
console.log(-3.14 >>> 0);     // 4294967293
console.log(NaN >>> 0);       // 0
console.log(Infinity >>> 0);  // 0
```

### Quick Comparison of Conversion Tricks

| Expression     | Result Type          | Truncation | Handles NaN  | Max Range          |
|----------------|----------------------|------------|-------------|---------------------|
| `~~x`          | Signed 32-bit        | Yes        | Returns 0   | -2^31 to 2^31 - 1  |
| `x \| 0`      | Signed 32-bit        | Yes        | Returns 0   | -2^31 to 2^31 - 1  |
| `x >>> 0`      | Unsigned 32-bit      | Yes        | Returns 0   | 0 to 2^32 - 1      |
| `Math.trunc(x)`| Number (64-bit float)| No         | Returns NaN | Full double range   |

---

## Math.clz32 -- The One Built-in Bit Function

JavaScript has exactly one built-in bit counting function: `Math.clz32()`, which counts the number of leading zero bits in the 32-bit representation.

```javascript
console.log(Math.clz32(1));          // 31 (0b0...0001)
console.log(Math.clz32(1024));       // 21 (0b0...010000000000)
console.log(Math.clz32(0));          // 32
console.log(Math.clz32(-1));         // 0  (0xFFFFFFFF, all bits set)
console.log(Math.clz32(0x80000000)); // 0  (MSB set)
console.log(Math.clz32(0x7FFFFFFF)); // 1  (all bits set except MSB)
```

---

## Manual Implementations: popcount, ctz, and More

JavaScript has no built-in `popcount` (count set bits) or `ctz` (count trailing zeros). Here are efficient implementations.

### popcount -- Count Set Bits

See [[Count Set Bits]] for the algorithm explanation.

```javascript
// Simple loop implementation
function popcount_simple(n) {
    n = n >>> 0;  // Convert to unsigned 32-bit
    let count = 0;
    while (n !== 0) {
        count += n & 1;
        n >>>= 1;
    }
    return count;
}

// Brian Kernighan's algorithm -- O(number of set bits)
function popcount_kernighan(n) {
    n = n >>> 0;
    let count = 0;
    while (n !== 0) {
        n &= (n - 1);  // Clear the lowest set bit
        count++;
    }
    return count;
}

// Parallel bit-counting (Hamming weight) -- constant time
// See [[Count Set Bits]] for detailed explanation
function popcount(n) {
    n = n >>> 0;
    n = n - ((n >>> 1) & 0x55555555);
    n = (n & 0x33333333) + ((n >>> 2) & 0x33333333);
    n = (n + (n >>> 4)) & 0x0F0F0F0F;
    return (n * 0x01010101) >>> 24;
}

console.log(popcount(0b10101010));  // 4
console.log(popcount(0xFFFFFFFF));  // 32
console.log(popcount(0));           // 0
```

### ctz -- Count Trailing Zeros

```javascript
function ctz(n) {
    n = n >>> 0;
    if (n === 0) return 32;
    // Isolate lowest set bit, then count leading zeros
    // clz32 of the isolated bit gives (31 - position)
    return 31 - Math.clz32(n & (-n >>> 0));
}

console.log(ctz(0b1000));   // 3
console.log(ctz(0b10100));  // 2
console.log(ctz(1));        // 0
console.log(ctz(0));        // 32
```

### Log2 -- Position of Highest Set Bit

```javascript
function log2(n) {
    n = n >>> 0;
    if (n === 0) return -1;  // undefined for 0
    return 31 - Math.clz32(n);
}

console.log(log2(1));    // 0
console.log(log2(8));    // 3
console.log(log2(255));  // 7
console.log(log2(256));  // 8
```

### isPowerOfTwo -- Power of Two Check

See [[Check Power of Two]] for the algorithm.

```javascript
function isPowerOfTwo(n) {
    n = n >>> 0;
    return n !== 0 && (n & (n - 1)) === 0;
}

console.log(isPowerOfTwo(64));   // true
console.log(isPowerOfTwo(65));   // false
console.log(isPowerOfTwo(0));    // false
console.log(isPowerOfTwo(1));    // true
```

---

## Precision Warnings and Edge Cases

### The 32-bit Boundary

```ad-warning
title: Values above 2^31 - 1 are dangerous with bitwise operators
Any bitwise operation on a number above 2,147,483,647 (2^31 - 1) or below -2,147,483,648 (-2^31) will produce unexpected results due to 32-bit truncation. Always be aware of your value ranges when using bitwise operators.
```

```javascript
// Looks correct but isn't:
let large = 0x1_0000_0000;  // 2^32 = 4294967296
console.log(large & large); // 0 (truncated to 32 bits, became 0)

// The mask you intended was 32 bits wide, but:
let mask = 0xFFFFFFFF;
console.log(mask);           // 4294967295 (correct as a Number)
console.log(mask | 0);       // -1 (as 32-bit signed!)
console.log(mask >>> 0);     // 4294967295 (as 32-bit unsigned -- correct)
```

### NaN and Non-Numeric Values

```javascript
// All non-numeric values become 0 in bitwise operations
console.log(undefined | 0);  // 0
console.log(null | 0);       // 0
console.log(NaN | 0);        // 0
console.log("hello" | 0);    // 0
console.log(true | 0);       // 1  (true -> 1)
console.log(false | 0);      // 0  (false -> 0)
console.log("42" | 0);       // 42 (string parsed as number first)
```

### Floating-Point Precision and Bitwise

```javascript
// Floating-point representation issues can affect bitwise ops
let a = 0.1 + 0.2;     // 0.30000000000000004 (IEEE 754 imprecision)
console.log(a | 0);     // 0 (fractional part dropped)

// Be careful with computed values that should be integers:
let computed = 1e10 / 3;  // 3333333333.3333335
console.log(computed | 0);  // -961633963 (truncated AND wrapped!)
```

---

## Complete Example: Binary Protocol Parser

Here is a practical example parsing a simple binary message format using typed arrays and bitwise operations:

```javascript
/**
 * Parse a 4-byte status message:
 * Byte 0: version (4 bits) | type (4 bits)
 * Byte 1: flags (8 individual bits)
 * Byte 2-3: payload length (16-bit unsigned, big-endian)
 */
function parseStatusMessage(buffer) {
    if (buffer.byteLength < 4) {
        throw new Error("Buffer too small");
    }

    const view = new DataView(buffer);
    const byte0 = view.getUint8(0);
    const byte1 = view.getUint8(1);
    const payloadLength = view.getUint16(2, false);  // big-endian

    // Extract version and type using masks -- see [[Creating Bit Masks]]
    const version = (byte0 >>> 4) & 0x0F;  // Upper nibble
    const type    = byte0 & 0x0F;           // Lower nibble

    // Extract individual flag bits -- see [[Check Set and Clear Bits]]
    const flags = {
        connected:  !!(byte1 & (1 << 0)),
        encrypted:  !!(byte1 & (1 << 1)),
        compressed: !!(byte1 & (1 << 2)),
        error:      !!(byte1 & (1 << 3)),
        priority:   (byte1 >>> 4) & 0x03,   // 2-bit priority level (bits 4-5)
        reserved:   (byte1 >>> 6) & 0x03,   // 2 reserved bits (bits 6-7)
    };

    return { version, type, flags, payloadLength };
}

// Create a test message
const buffer = new ArrayBuffer(4);
const writer = new DataView(buffer);

// Version 3, Type 5
writer.setUint8(0, (3 << 4) | 5);  // 0b0011_0101 = 0x35

// Flags: connected, encrypted, priority=2
writer.setUint8(1, (1 << 0) | (1 << 1) | (2 << 4));  // 0b0010_0011 = 0x23

// Payload length: 1024 (big-endian)
writer.setUint16(2, 1024, false);

const result = parseStatusMessage(buffer);
console.log(result);
// {
//   version: 3,
//   type: 5,
//   flags: {
//     connected: true,
//     encrypted: true,
//     compressed: false,
//     error: false,
//     priority: 2,
//     reserved: 0
//   },
//   payloadLength: 1024
// }
```

---

## Complete Example: Bit Manipulation Utility Class

```javascript
/**
 * Utility class for 32-bit bitwise operations in JavaScript.
 * All methods work with unsigned 32-bit integers.
 */
class Bits32 {
    /**
     * Count set bits (popcount) -- see [[Count Set Bits]]
     */
    static popcount(n) {
        n = n >>> 0;
        n = n - ((n >>> 1) & 0x55555555);
        n = (n & 0x33333333) + ((n >>> 2) & 0x33333333);
        n = (n + (n >>> 4)) & 0x0F0F0F0F;
        return (n * 0x01010101) >>> 24;
    }

    /** Count leading zeros */
    static clz(n) { return Math.clz32(n >>> 0); }

    /** Count trailing zeros */
    static ctz(n) {
        n = n >>> 0;
        if (n === 0) return 32;
        return 31 - Math.clz32(n & (-n >>> 0));
    }

    /** Check if power of two -- see [[Check Power of Two]] */
    static isPow2(n) {
        n = n >>> 0;
        return n !== 0 && (n & (n - 1)) === 0;
    }

    /** Round up to next power of two */
    static nextPow2(n) {
        n = n >>> 0;
        if (n === 0) return 1;
        n--;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n + 1) >>> 0;
    }

    /** Check if bit at position is set -- see [[Check Set and Clear Bits]] */
    static test(n, pos)  { return ((n >>> 0) & (1 << pos)) !== 0; }

    /** Set bit at position -- see [[Check Set and Clear Bits]] */
    static set(n, pos)   { return ((n >>> 0) | (1 << pos)) >>> 0; }

    /** Clear bit at position -- see [[Check Set and Clear Bits]] */
    static clear(n, pos) { return ((n >>> 0) & ~(1 << pos)) >>> 0; }

    /** Toggle bit at position -- see [[Toggle Bits]] */
    static toggle(n, pos){ return ((n >>> 0) ^ (1 << pos)) >>> 0; }

    /** Display as binary string, grouped by nibble */
    static toBinary(n, bits = 32) {
        n = n >>> 0;
        let str = n.toString(2).padStart(bits, '0');
        return str.match(/.{1,4}/g).join('_');
    }
}

// Usage:
console.log(Bits32.popcount(0xFF));     // 8
console.log(Bits32.clz(0x00FF));        // 24
console.log(Bits32.ctz(0x0100));        // 8
console.log(Bits32.isPow2(256));        // true
console.log(Bits32.nextPow2(100));      // 128
console.log(Bits32.test(0b1010, 1));    // true
console.log(Bits32.toBinary(0xDEAD));   // "0000_0000_0000_0000_1101_1110_1010_1101"
```

---

## Cross-Language Comparison

| Feature                      | JavaScript                  | C#                          | C++                         |
|------------------------------|-----------------------------|-----------------------------|---------------------------  |
| Number type                  | 64-bit float                | int, long, etc.             | int, int32_t, etc.          |
| Bitwise operand width        | 32-bit signed               | Type's own width            | Type's own width            |
| Unsigned right shift op      | `>>>` (built-in)            | `>>>` (C# 11+)             | None (cast to unsigned)     |
| Built-in popcount            | None                        | `BitOperations.PopCount`    | `std::popcount` (C++20)     |
| Built-in CLZ                 | `Math.clz32`                | `BitOperations.LeadingZeroCount` | `std::countl_zero` (C++20) |
| Built-in CTZ                 | None                        | `BitOperations.TrailingZeroCount`| `std::countr_zero` (C++20) |
| Arbitrary precision          | `BigInt`                    | `BigInteger`                | None (use libraries)        |
| Binary data API              | `DataView`, typed arrays    | `BinaryPrimitives`, `Span`  | `std::bit_cast`, raw pointers|
| Endianness control           | `DataView` methods          | `BinaryPrimitives`          | `std::endian` (C++20)       |
| Undefined behavior risk      | None (JS is memory-safe)    | None (managed runtime)      | Yes (shifts, overflow)      |

---

## Quick Reference Summary

| Feature                        | Availability     | Notes                                              |
|--------------------------------|------------------|----------------------------------------------------|
| Binary literals (`0b`)        | ES2015+          | No digit separators until ES2021                   |
| Hex literals (`0x`)           | Always           |                                                    |
| Numeric separators (`_`)      | ES2021+          | Works in binary, hex, octal, decimal               |
| `>>>`                         | Always           | The only unsigned-result bitwise op                |
| `Math.clz32`                  | ES2015+          | The only built-in bit counting function            |
| Typed arrays                  | ES2015+          | `Uint8Array`, `Int32Array`, etc.                   |
| `DataView`                    | ES2015+          | Explicit endianness control                        |
| `BigInt`                      | ES2020+          | Arbitrary precision, cannot mix with Number         |
| `BigInt64Array`               | ES2020+          | 64-bit integer typed arrays                        |
| `BigUint64Array`              | ES2020+          | 64-bit unsigned integer typed arrays               |

See also: [[Bit Manipulation in CSharp]] for the C# perspective (guaranteed types, rich standard library), and [[Bit Manipulation in CPP]] for C++'s power and pitfalls.
