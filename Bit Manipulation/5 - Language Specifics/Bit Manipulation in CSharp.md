---
tags:
  - bit-manipulation
  - csharp
  - language-specific
---

## Overview

C# provides a rich set of tools for bit manipulation, from low-level bitwise operators to high-level classes like `BitArray` and the modern `System.Numerics.BitOperations` class. This note covers every C#-specific feature related to working with bits, organized from fundamentals through advanced APIs.

---

## Integer Types and Sizes

C# has a complete set of integral types with well-defined, platform-independent sizes. Unlike C/C++, the sizes are guaranteed by the language specification.

| Type     | Signed? | Bits | Min Value                    | Max Value                     |
|----------|---------|------|------------------------------|-------------------------------|
| `sbyte`  | Yes     | 8    | -128                         | 127                           |
| `byte`   | No      | 8    | 0                            | 255                           |
| `short`  | Yes     | 16   | -32,768                      | 32,767                        |
| `ushort` | No      | 16   | 0                            | 65,535                        |
| `int`    | Yes     | 32   | -2,147,483,648               | 2,147,483,647                 |
| `uint`   | No      | 32   | 0                            | 4,294,967,295                 |
| `long`   | Yes     | 64   | -9,223,372,036,854,775,808   | 9,223,372,036,854,775,807     |
| `ulong`  | No      | 64   | 0                            | 18,446,744,073,709,551,615    |
| `nint`   | Yes     | *    | Platform-dependent           | Platform-dependent            |
| `nuint`  | No      | *    | 0                            | Platform-dependent            |

`nint` and `nuint` (C# 9+) are native-sized integers -- 32 bits on a 32-bit platform, 64 bits on a 64-bit platform. They are primarily used for interop and pointer arithmetic.

```ad-tip
title: Guaranteed sizes
Unlike C/C++ where `int` might be 16, 32, or 64 bits depending on the platform, C#'s `int` is always 32 bits. This makes bit manipulation code fully portable. See [[Bits Bytes and Words]] for more on data unit sizes and [[Signed and Unsigned Integers]] for signed vs unsigned representation.
```

---

## Binary and Hexadecimal Literals

C# 7.0 introduced binary literals and digit separators, making it easy to write and read bit patterns directly in code.

### Binary Literals (`0b` prefix)

```csharp
int flags = 0b1010_1100;     // 172 in decimal
byte mask = 0b0000_1111;     // Lower nibble mask
long big  = 0b1100_0011_1010_0101_0000_1111_1111_0000;
```

### Hexadecimal Literals (`0x` prefix)

```csharp
int color = 0xFF_00_FF;      // Magenta (RGB)
uint addr = 0xDEAD_BEEF;     // Classic debug pattern
byte high = 0xF0;            // Upper nibble mask
```

### Digit Separators (`_`)

The underscore `_` can be placed between any digits for readability. The compiler ignores them entirely.

```csharp
// Group by nibble (4 bits) for readability
int grouped = 0b1010_0011_1100_0101;

// Group hex by byte
uint color = 0xFF_A0_B0_C0;

// Also works in decimal
int million = 1_000_000;
```

```ad-tip
title: Readability convention
When writing binary literals, group bits into nibbles (4-bit groups) separated by underscores. This makes it trivial to convert to/from hexadecimal since each nibble maps to one hex digit.
```

---

## Bitwise Operators

C# supports all standard bitwise operators. They work on `int`, `uint`, `long`, `ulong`, and smaller integer types (which are promoted to `int` before the operation).

| Operator | Name               | Example        | Link                         |
|----------|--------------------|----------------|------------------------------|
| `&`      | Bitwise AND        | `a & b`        | [[AND Operator]]             |
| `\|`     | Bitwise OR         | `a \| b`       | [[OR Operator]]              |
| `^`      | Bitwise XOR        | `a ^ b`        | [[XOR Operator]]             |
| `~`      | Bitwise NOT        | `~a`           | [[NOT Operator]]             |
| `<<`     | Left shift         | `a << 3`       | [[Left Shift]]               |
| `>>`     | Right shift        | `a >> 3`       | [[Right Shift]]              |
| `>>>`    | Unsigned right shift (C# 11+) | `a >>> 3` | [[Unsigned Right Shift]] |

### Compound Assignment Operators

All binary bitwise operators have compound assignment forms:

```csharp
int flags = 0b1010;
flags &= 0b1100;   // flags = flags & 0b1100  --> 0b1000
flags |= 0b0011;   // flags = flags | 0b0011  --> 0b1011
flags ^= 0b0101;   // flags = flags ^ 0b0101  --> 0b1110
flags <<= 2;        // flags = flags << 2      --> 0b111000
flags >>= 1;        // flags = flags >> 1      --> 0b11100
```

### Operator Behavior by Type

```csharp
// For signed types, >> performs arithmetic right shift (preserves sign bit)
int negative = -8;                // 0xFFFF_FFF8
int shifted = negative >> 2;      // -2 (0xFFFF_FFFE) -- sign bit preserved

// For unsigned types, >> performs logical right shift (fills with 0)
uint unsignedVal = 0xFFFF_FFF8;
uint uShifted = unsignedVal >> 2; // 0x3FFF_FFFE -- zero-filled

// >>> always performs logical (unsigned) right shift, regardless of type
int logicalShift = negative >>> 2; // 0x3FFF_FFFE -- zero-filled even though int is signed
```

```ad-warning
title: Unsigned right shift (>>>) is C# 11+ only
The `>>>` operator was added in C# 11 (.NET 7). In earlier versions, to perform an unsigned right shift on a signed integer, you had to cast to unsigned first: `(int)((uint)value >> n)`. See [[Unsigned Right Shift]] for more details.
```

### Promotion Rules

```ad-note
title: Integral promotion
When you apply bitwise operators to `byte`, `sbyte`, `short`, or `ushort`, C# automatically promotes the operands to `int` before performing the operation. This means the result is also `int`, and you need an explicit cast to assign it back to a smaller type.
```

```csharp
byte a = 0b1010_0101;
byte b = 0b1100_0011;

// This won't compile without the cast:
byte result = (byte)(a & b);  // Must cast back to byte

// The intermediate result is int:
int intermediate = a & b;     // This works without casting
```

---

## System.Numerics.BitOperations

The `System.Numerics.BitOperations` static class (introduced in .NET Core 3.0 / .NET 5+) provides hardware-accelerated bit manipulation methods. These compile down to single CPU instructions when available (e.g., `POPCNT`, `LZCNT`, `TZCNT` on x86).

```csharp
using System.Numerics;
```

### PopCount -- Count Set Bits

Returns the number of `1` bits in an integer. See [[Count Set Bits]] for the algorithm.

```csharp
uint value = 0b1010_1100_0011_0101;
int count = BitOperations.PopCount(value);  // 8

ulong big = 0xFFFF_FFFF_0000_0000;
int bigCount = BitOperations.PopCount(big); // 32
```

### LeadingZeroCount and TrailingZeroCount

Count the number of consecutive zero bits from the MSB or LSB.

```csharp
uint value = 0b0000_0000_0000_0000_0010_0000_0000_0000;  // bit 13 set
int leading  = BitOperations.LeadingZeroCount(value);     // 18
int trailing = BitOperations.TrailingZeroCount(value);    // 13

// Special case: all zeros
int allZero = BitOperations.LeadingZeroCount((uint)0);    // 32
int allZeroT = BitOperations.TrailingZeroCount((uint)0);  // 32
```

### Log2 -- Position of Highest Set Bit

Returns the base-2 logarithm (floor), which equals the position of the highest set bit.

```csharp
uint value = 0b0001_0000;  // 16
int log = BitOperations.Log2(value);  // 4 (bit 4 is the highest set bit)

uint val2 = 255;  // 0b1111_1111
int log2 = BitOperations.Log2(val2);  // 7
```

### RotateLeft and RotateRight -- Circular Rotations

Bits that "fall off" one end wrap around to the other.

```csharp
uint value = 0b1010_0000_0000_0000_0000_0000_0000_0011;

uint rotL = BitOperations.RotateLeft(value, 4);
// 0b0000_0000_0000_0000_0000_0000_0011_1010

uint rotR = BitOperations.RotateRight(value, 4);
// 0b0011_1010_0000_0000_0000_0000_0000_0000
```

### IsPow2 -- Power of Two Check

Returns `true` if the value is a power of two. See [[Check Power of Two]] for the bit trick behind this.

```csharp
bool yes = BitOperations.IsPow2(64);   // true  (0b0100_0000)
bool no  = BitOperations.IsPow2(65);   // false (0b0100_0001)
bool zero = BitOperations.IsPow2(0);   // false (0 is not a power of 2)
```

### RoundUpToPowerOf2

Rounds a value up to the nearest power of 2 (or returns the value itself if already a power of 2).

```csharp
uint result1 = BitOperations.RoundUpToPowerOf2(100);  // 128
uint result2 = BitOperations.RoundUpToPowerOf2(128);  // 128
uint result3 = BitOperations.RoundUpToPowerOf2(1);    // 1
```

```ad-tip
title: Hardware acceleration
On modern x86 processors, `PopCount` uses the `POPCNT` instruction, `LeadingZeroCount` uses `LZCNT`, and `TrailingZeroCount` uses `TZCNT`. On ARM, equivalent instructions are used. These single-instruction implementations are dramatically faster than manual bit-counting loops. Always prefer `BitOperations` over hand-rolled implementations.
```

---

## BitConverter Class

`BitConverter` converts between primitive types and their byte-level representations. This is essential when working with binary protocols, file formats, or network data.

### Basic Conversions

```csharp
// int to bytes
int value = 0x0A0B0C0D;
byte[] bytes = BitConverter.GetBytes(value);
// On little-endian: [0x0D, 0x0C, 0x0B, 0x0A]
// On big-endian:    [0x0A, 0x0B, 0x0C, 0x0D]

// bytes back to int
int restored = BitConverter.ToInt32(bytes, 0);  // 0x0A0B0C0D

// Works for all primitive types
byte[] doubleBytes = BitConverter.GetBytes(3.14);
double restored2 = BitConverter.ToDouble(doubleBytes, 0);

byte[] longBytes = BitConverter.GetBytes(0x0102030405060708L);
long restored3 = BitConverter.ToInt64(longBytes, 0);
```

### Checking Endianness

```csharp
if (BitConverter.IsLittleEndian)
{
    Console.WriteLine("This system is little-endian");
    // x86/x64 and most ARM are little-endian
}
else
{
    Console.WriteLine("This system is big-endian");
}
```

See [[Endianness]] for a full discussion of byte ordering.

### Viewing Bit Representation

```csharp
// Useful for debugging: see the hex representation
float pi = 3.14f;
byte[] piBytes = BitConverter.GetBytes(pi);
string hex = BitConverter.ToString(piBytes);
Console.WriteLine(hex);  // "C3-F5-48-40" (little-endian IEEE 754)
```

```ad-warning
title: Endianness matters
`BitConverter` uses the system's native endianness. If you're building cross-platform binary protocols, you should use `BinaryPrimitives` (see below) instead, which lets you specify the endianness explicitly.
```

---

## BinaryPrimitives and Span&lt;byte&gt;

`System.Buffers.Binary.BinaryPrimitives` (introduced in .NET Core 2.1) is the modern, allocation-free way to read/write integers from byte buffers with explicit endianness control.

```csharp
using System.Buffers.Binary;
```

### Writing with Explicit Endianness

```csharp
byte[] buffer = new byte[8];
Span<byte> span = buffer;

// Write a 32-bit integer in big-endian (network byte order)
BinaryPrimitives.WriteInt32BigEndian(span, 0x0A0B0C0D);
// buffer: [0x0A, 0x0B, 0x0C, 0x0D, 0x00, 0x00, 0x00, 0x00]

// Write a 32-bit integer in little-endian
BinaryPrimitives.WriteInt32LittleEndian(span.Slice(4), 0x01020304);
// buffer: [0x0A, 0x0B, 0x0C, 0x0D, 0x04, 0x03, 0x02, 0x01]
```

### Reading with Explicit Endianness

```csharp
byte[] networkData = { 0x0A, 0x0B, 0x0C, 0x0D };
ReadOnlySpan<byte> data = networkData;

int bigEndian = BinaryPrimitives.ReadInt32BigEndian(data);
// 0x0A0B0C0D = 168496141

int littleEndian = BinaryPrimitives.ReadInt32LittleEndian(data);
// 0x0D0C0B0A = 219025162
```

### Available Methods

Methods exist for all integer sizes and both endiannesses:

| Read Method                          | Write Method                          |
|--------------------------------------|---------------------------------------|
| `ReadInt16BigEndian`                 | `WriteInt16BigEndian`                 |
| `ReadInt16LittleEndian`             | `WriteInt16LittleEndian`             |
| `ReadInt32BigEndian`                 | `WriteInt32BigEndian`                 |
| `ReadInt32LittleEndian`             | `WriteInt32LittleEndian`             |
| `ReadInt64BigEndian`                 | `WriteInt64BigEndian`                 |
| `ReadInt64LittleEndian`             | `WriteInt64LittleEndian`             |
| `ReadUInt16BigEndian`               | `WriteUInt16BigEndian`               |
| `ReadUInt16LittleEndian`           | `WriteUInt16LittleEndian`           |
| `ReadUInt32BigEndian`               | `WriteUInt32BigEndian`               |
| `ReadUInt32LittleEndian`           | `WriteUInt32LittleEndian`           |
| `ReadUInt64BigEndian`               | `WriteUInt64BigEndian`               |
| `ReadUInt64LittleEndian`           | `WriteUInt64LittleEndian`           |

```ad-tip
title: BinaryPrimitives vs BitConverter
Prefer `BinaryPrimitives` over `BitConverter` when:
- You need explicit endianness control
- You want to avoid heap allocations (works directly on `Span<byte>`)
- You are writing performance-sensitive code (no byte array creation)

`BitConverter` is fine for quick conversions where allocation is not a concern.
```

---

## BitArray Class

`System.Collections.BitArray` provides a dynamically-sized array of boolean values stored as individual bits. It supports bulk bitwise operations on entire arrays.

### Creating a BitArray

```csharp
using System.Collections;

// From size (all false/0)
BitArray bits1 = new BitArray(16);  // 16 bits, all false

// From boolean array
BitArray bits2 = new BitArray(new bool[] { true, false, true, true });

// From byte array (each byte = 8 bits, LSB first within each byte)
BitArray bits3 = new BitArray(new byte[] { 0b1010_0101 });
// bits3[0] = true (bit 0 of 0xA5), bits3[7] = true (bit 7 of 0xA5)

// From int array (each int = 32 bits)
BitArray bits4 = new BitArray(new int[] { 0xFF });

// All bits set to a specific value
BitArray bits5 = new BitArray(8, true);  // 8 bits, all true
```

### Accessing and Modifying Bits

```csharp
BitArray flags = new BitArray(8);

// Set individual bits
flags.Set(0, true);   // Set bit 0
flags.Set(3, true);   // Set bit 3
flags[5] = true;      // Indexer also works

// Read individual bits
bool bit0 = flags.Get(0);  // true
bool bit1 = flags[1];      // false

// Number of bits
int length = flags.Length;  // 8
int count = flags.Count;    // 8 (same as Length)
```

### Bulk Bitwise Operations

The power of `BitArray` is performing operations on entire arrays at once. These are analogous to scalar [[AND Operator]], [[OR Operator]], [[XOR Operator]], and [[NOT Operator]], but applied element-wise across the entire array.

```csharp
BitArray a = new BitArray(new byte[] { 0b1010_1010 });
BitArray b = new BitArray(new byte[] { 0b1100_1100 });

// AND -- modifies 'a' in-place and returns it
BitArray andResult = a.And(b);
// Result: 0b1000_1000

// Reset for next example
a = new BitArray(new byte[] { 0b1010_1010 });

// OR
BitArray orResult = a.Or(b);
// Result: 0b1110_1110

// Reset
a = new BitArray(new byte[] { 0b1010_1010 });

// XOR
BitArray xorResult = a.Xor(b);
// Result: 0b0110_0110

// NOT (unary, no second operand)
a = new BitArray(new byte[] { 0b1010_1010 });
BitArray notResult = a.Not();
// Result: 0b0101_0101
```

```ad-warning
title: In-place modification
`And()`, `Or()`, `Xor()`, and `Not()` all modify the BitArray in-place AND return the same reference. If you need to preserve the original, clone it first: `BitArray copy = new BitArray(original);`
```

### Converting Back to Bytes

```csharp
BitArray bits = new BitArray(new byte[] { 0b1010_0101, 0b1100_0011 });

byte[] result = new byte[(bits.Length + 7) / 8];  // Ceiling division
bits.CopyTo(result, 0);
// result: [0xA5, 0xC3]
```

---

## [Flags] Enum Attribute

The `[Flags]` attribute marks an enum as a bit field, where each value represents a single bit (a power of 2). This allows combining multiple values with bitwise OR. See [[Flag Enums and Bit Flags]] for the general concept.

### Declaring a Flags Enum

```csharp
[Flags]
public enum FilePermissions
{
    None    = 0,         // 0b0000_0000
    Read    = 1 << 0,    // 0b0000_0001
    Write   = 1 << 1,    // 0b0000_0010
    Execute = 1 << 2,    // 0b0000_0100
    Delete  = 1 << 3,    // 0b0000_1000

    // Convenience combinations
    ReadWrite      = Read | Write,           // 0b0000_0011
    ReadExecute    = Read | Execute,         // 0b0000_0101
    All            = Read | Write | Execute | Delete  // 0b0000_1111
}
```

### Using Flags with Bitwise Operations

```csharp
// Combine flags
FilePermissions perms = FilePermissions.Read | FilePermissions.Write;

// Check if a flag is set (manual way -- uses AND, see [[AND Operator]])
bool canRead = (perms & FilePermissions.Read) == FilePermissions.Read;  // true
bool canExec = (perms & FilePermissions.Execute) == FilePermissions.Execute;  // false

// Check if a flag is set (HasFlag method)
bool canRead2 = perms.HasFlag(FilePermissions.Read);     // true
bool canExec2 = perms.HasFlag(FilePermissions.Execute);  // false

// Add a flag (uses OR, see [[OR Operator]])
perms |= FilePermissions.Execute;
// perms = Read | Write | Execute

// Remove a flag (uses AND + NOT, see [[Creating Bit Masks]])
perms &= ~FilePermissions.Write;
// perms = Read | Execute

// Toggle a flag (uses XOR, see [[XOR Operator]])
perms ^= FilePermissions.Delete;
// perms = Read | Execute | Delete
```

### ToString with [Flags]

```csharp
FilePermissions perms = FilePermissions.Read | FilePermissions.Write | FilePermissions.Execute;
Console.WriteLine(perms);
// Output: "Read, Write, Execute" (the [Flags] attribute makes ToString show all set flags)

// Without [Flags], it would show "7" (the numeric value)
```

```ad-tip
title: HasFlag performance
In older .NET Framework versions, `HasFlag()` caused a boxing allocation. Since .NET Core 2.1+, the JIT recognizes and optimizes `HasFlag()` to a simple AND comparison with no boxing. In modern .NET, there is no performance penalty.
```

```ad-warning
title: Always use powers of 2
Each individual flag value must be a power of 2 (1, 2, 4, 8, 16...). If you use sequential values (0, 1, 2, 3, 4...), the bitwise operations will produce nonsensical results. The `1 << n` pattern is the clearest way to define flag values.
```

---

## Checked and Unchecked Contexts

C# allows you to control overflow behavior for arithmetic and conversion operations. This can interact with bit manipulation when shifting or combining values.

### Default Behavior (Unchecked)

By default, integer overflow wraps around silently (for performance):

```csharp
int max = int.MaxValue;     // 2,147,483,647
int overflow = max + 1;      // -2,147,483,648 (wraps around, no exception)

uint umax = uint.MaxValue;  // 4,294,967,295
uint uoverflow = umax + 1;  // 0 (wraps around)
```

### Checked Context

In a `checked` context, overflow throws `System.OverflowException`:

```csharp
checked
{
    int max = int.MaxValue;
    // int overflow = max + 1;  // Throws OverflowException!
}

// Or per-expression:
// int result = checked(int.MaxValue + 1);  // Throws OverflowException
```

### Unchecked Context (Explicit)

Force unchecked behavior even if the project is compiled with `/checked+`:

```csharp
unchecked
{
    int max = int.MaxValue;
    int wrap = max + 1;  // -2,147,483,648 (no exception, guaranteed)
}
```

### Bitwise Operations and Checked/Unchecked

```ad-note
title: Bitwise operators are unaffected
The bitwise operators (`&`, `|`, `^`, `~`, `<<`, `>>`, `>>>`) themselves never throw overflow exceptions, even in a `checked` context. Overflow checking only applies to arithmetic operators (`+`, `-`, `*`, `/`) and explicit conversions (casts between integer types). Shift operations simply discard bits that go beyond the type's width.
```

```csharp
checked
{
    int a = int.MaxValue;
    int b = a << 1;      // No exception -- bits just shift (result: -2)
    int c = ~a;           // No exception -- bitwise NOT is always safe
    int d = a & 0xFF;     // No exception -- AND is always safe
}
```

### Unchecked Conversions in Bit Manipulation

A common use case for `unchecked` is converting between signed and unsigned types when you care about the bit pattern, not the numeric value:

```csharp
// Reinterpret bit pattern: treat -1 as its unsigned representation
int signed = -1;                          // All bits set: 0xFFFF_FFFF
uint asUnsigned = unchecked((uint)signed); // 4,294,967,295

// Or the reverse
uint big = 0xFFFF_FFFF;
int asSigned = unchecked((int)big);       // -1
```

---

## Convert Class -- Binary String Conversions

The `System.Convert` class provides methods to convert between integers and their string representations in various bases.

### Integer to Binary/Hex String

```csharp
int value = 42;

// To binary string
string binary = Convert.ToString(value, 2);   // "101010"

// Padded to specific width
string padded = Convert.ToString(value, 2).PadLeft(8, '0');  // "00101010"
string padded16 = Convert.ToString(value, 2).PadLeft(16, '0');  // "0000000000101010"

// To hex string
string hex = Convert.ToString(value, 16);  // "2a"
string hexUpper = value.ToString("X8");     // "0000002A"

// Negative numbers show two's complement representation
string neg = Convert.ToString(-1, 2);
// "11111111111111111111111111111111" (32 ones)
```

### Binary/Hex String to Integer

```csharp
// From binary string
int fromBin = Convert.ToInt32("10101010", 2);   // 170
long fromBinL = Convert.ToInt64("1010101010101010", 2);

// From hex string
int fromHex = Convert.ToInt32("FF", 16);        // 255
int fromHex2 = Convert.ToInt32("DeadBeef", 16); // throws OverflowException (too large for int)
uint fromHexU = Convert.ToUInt32("DeadBeef", 16); // 3735928559
```

```ad-tip
title: Debugging helper
When debugging bit manipulation code, printing values in binary is invaluable. Consider creating a helper extension method:
```

```csharp
public static class BitExtensions
{
    public static string ToBinaryString(this int value, int width = 32)
        => Convert.ToString(value, 2).PadLeft(width, '0');

    public static string ToBinaryString(this uint value, int width = 32)
        => Convert.ToString((int)value, 2).PadLeft(width, '0');

    public static string ToBinaryGrouped(this int value, int groupSize = 4)
    {
        string binary = Convert.ToString(value, 2).PadLeft(32, '0');
        return string.Join("_", Enumerable.Range(0, 32 / groupSize)
            .Select(i => binary.Substring(i * groupSize, groupSize)));
    }
}

// Usage:
int val = 0xDEAD;
Console.WriteLine(val.ToBinaryString());
// "00000000000000001101111010101101"
Console.WriteLine(val.ToBinaryGrouped());
// "0000_0000_0000_0000_1101_1110_1010_1101"
```

---

## Complete Example: Combining C# Bit Manipulation Features

Here is a practical example that ties together multiple features -- implementing a simple permission system.

```csharp
using System;
using System.Numerics;

[Flags]
public enum Permissions : uint
{
    None       = 0,
    Read       = 1u << 0,
    Write      = 1u << 1,
    Execute    = 1u << 2,
    Delete     = 1u << 3,
    Admin      = 1u << 4,
    CreateUser = 1u << 5,
    Audit      = 1u << 6,
    Backup     = 1u << 7,
}

public class PermissionManager
{
    private uint _permissions;

    public PermissionManager(Permissions initial = Permissions.None)
    {
        _permissions = (uint)initial;
    }

    // Grant permissions (OR -- see [[OR Operator]])
    public void Grant(Permissions perms)
        => _permissions |= (uint)perms;

    // Revoke permissions (AND + NOT -- see [[Creating Bit Masks]])
    public void Revoke(Permissions perms)
        => _permissions &= ~(uint)perms;

    // Toggle permissions (XOR -- see [[XOR Operator]])
    public void Toggle(Permissions perms)
        => _permissions ^= (uint)perms;

    // Check if ALL specified permissions are set
    public bool HasAll(Permissions perms)
        => (_permissions & (uint)perms) == (uint)perms;

    // Check if ANY of the specified permissions are set
    public bool HasAny(Permissions perms)
        => (_permissions & (uint)perms) != 0;

    // Count how many permissions are active (uses PopCount)
    public int ActiveCount()
        => BitOperations.PopCount(_permissions);

    // Display current permissions in binary
    public override string ToString()
        => $"Permissions: {(Permissions)_permissions} (0b{Convert.ToString(_permissions, 2).PadLeft(8, '0')})";
}

// Usage:
var pm = new PermissionManager();
pm.Grant(Permissions.Read | Permissions.Write);
Console.WriteLine(pm);
// Permissions: Read, Write (0b00000011)

Console.WriteLine(pm.HasAll(Permissions.Read | Permissions.Write));  // true
Console.WriteLine(pm.HasAny(Permissions.Execute));                   // false
Console.WriteLine($"Active permissions: {pm.ActiveCount()}");        // 2

pm.Toggle(Permissions.Write | Permissions.Execute);
Console.WriteLine(pm);
// Permissions: Read, Execute (0b00000101)
```

---

## Quick Reference Summary

| Feature                          | Namespace / Class                     | .NET Version          |
|----------------------------------|---------------------------------------|-----------------------|
| Binary literals (`0b`)          | Language feature                      | C# 7.0+              |
| Digit separators (`_`)          | Language feature                      | C# 7.0+              |
| `>>>` operator                  | Language feature                      | C# 11+ (.NET 7+)     |
| `BitOperations`                 | `System.Numerics`                     | .NET Core 3.0+        |
| `BitConverter`                  | `System`                              | .NET Framework 1.0+   |
| `BinaryPrimitives`              | `System.Buffers.Binary`               | .NET Core 2.1+        |
| `BitArray`                      | `System.Collections`                  | .NET Framework 1.0+   |
| `[Flags]` attribute             | `System`                              | .NET Framework 1.0+   |
| `checked`/`unchecked`           | Language feature                      | C# 1.0+              |

See also: [[Bit Manipulation in CPP]] for the C++ perspective, and [[Bit Manipulation in JavaScript]] for JavaScript's unique 32-bit conversion behavior.
