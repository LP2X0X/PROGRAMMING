---
tags:
  - bit-manipulation
  - endianness
  - memory
---

## What Is Endianness?

Endianness describes the **byte order** used to store multi-byte values in memory. When a value is wider than one byte (e.g., a 32-bit integer), the CPU must decide which byte to store at the lowest memory address.

There are two conventions:

| Endianness     | First Byte Stored (lowest address) | Mnemonic                  |
|----------------|-------------------------------------|--------------------------|
| **Big-endian**    | Most significant byte (MSB first)    | "Big end first"          |
| **Little-endian** | Least significant byte (LSB first)   | "Little end first"       |

```ad-note
title: Origin of the name
The terms come from Jonathan Swift's *Gulliver's Travels*, where two factions fought over which end of a boiled egg to crack open — the "big end" or the "little end." Danny Cohen applied the metaphor to byte ordering in his 1980 paper "On Holy Wars and a Plea for Peace."
```

---

## Big-Endian vs Little-Endian

Consider the 32-bit hexadecimal value `0x12345678`:

### Big-Endian (MSB First)

The most significant byte (`0x12`) is stored at the lowest memory address. Bytes appear in memory in the same order you would write them on paper.

```
Address:   0x00   0x01   0x02   0x03
          +------+------+------+------+
Value:    | 0x12 | 0x34 | 0x56 | 0x78 |
          +------+------+------+------+
            MSB                   LSB
            (most                 (least
           significant)          significant)
```

### Little-Endian (LSB First)

The least significant byte (`0x78`) is stored at the lowest memory address. Bytes appear in memory in **reverse** of the written order.

```
Address:   0x00   0x01   0x02   0x03
          +------+------+------+------+
Value:    | 0x78 | 0x56 | 0x34 | 0x12 |
          +------+------+------+------+
            LSB                   MSB
            (least                (most
           significant)          significant)
```

---

## Side-by-Side Comparison

For the value `0xDEADBEEF`:

```
           Address:  0x00   0x01   0x02   0x03

Big-endian:        | 0xDE | 0xAD | 0xBE | 0xEF |
                     MSB                   LSB

Little-endian:     | 0xEF | 0xBE | 0xAD | 0xDE |
                     LSB                   MSB
```

For the 16-bit value `0x0100` (decimal 256):

```
           Address:  0x00   0x01

Big-endian:        | 0x01 | 0x00 |     Reads naturally as "256"
Little-endian:     | 0x00 | 0x01 |     Looks like "1" if you read left-to-right
```

---

## Which Architectures Use Which?

| Architecture / System     | Endianness       |
|---------------------------|-----------------|
| x86 / x64 (Intel, AMD)   | Little-endian    |
| ARM (default mode)        | Little-endian (bi-endian capable) |
| MIPS                      | Configurable (big or little) |
| PowerPC                   | Big-endian (older), bi-endian (newer) |
| SPARC                     | Big-endian       |
| Network protocols (TCP/IP)| Big-endian ("network byte order") |
| Java class files          | Big-endian       |
| USB                       | Little-endian    |
| BMP file format           | Little-endian    |
| JPEG, PNG, GIF            | Big-endian       |

```ad-tip
title: Most common today
If you're developing on a modern desktop or server (Intel/AMD x64, or ARM-based Mac/phone), you're almost certainly on **little-endian**. Big-endian is primarily encountered in **network protocols** and certain **file formats**.
```

---

## Why Does Endianness Matter?

### 1. Network Communication

Network protocols use big-endian ("network byte order"). If your CPU is little-endian, you must convert byte order when sending/receiving multi-byte values.

```
Your little-endian machine stores port 80 (0x0050) as:

Memory: | 0x50 | 0x00 |

The network expects big-endian:

Wire:   | 0x00 | 0x50 |

Without conversion, the other end sees port 20480 (0x5000)!
```

### 2. File Formats

Binary file formats specify an endianness. Reading a BMP file (little-endian) on a big-endian system requires byte swapping.

### 3. Cross-Platform Data Exchange

Serialized binary data (structs written to disk, shared memory between processes on different architectures) must account for endianness differences.

### 4. Debugging and Hex Dumps

When inspecting raw memory in a debugger, the byte order can be confusing:

```
Variable x = 0x12345678 on a little-endian system

Memory dump at &x:
78 56 34 12

It looks "backwards" compared to the logical value.
This is normal for little-endian.
```

```ad-warning
title: Endianness applies to bytes, not bits
Endianness describes the order of **bytes** within a multi-byte value, NOT the order of **bits** within a byte. Bit ordering within a single byte is the same on all platforms (MSB at position 7, LSB at position 0). Bit-level ordering only matters in serialization contexts like network bit streams.
```

---

## Advantages of Each

### Little-Endian Advantages

- **Casting is free**: Reading the first 1, 2, or 4 bytes of a multi-byte value at the same address gives the correct truncated value. A 32-bit value `0x00000042` stored at address `p`: reading 1 byte at `p` gives `0x42`, reading 2 bytes gives `0x0042` — no address adjustment needed.

```
Address:  p     p+1   p+2   p+3
         | 0x42 | 0x00 | 0x00 | 0x00 |   (0x00000042, little-endian)

Read 1 byte at p:  0x42       (correct as uint8)
Read 2 bytes at p: 0x0042     (correct as uint16)
Read 4 bytes at p: 0x00000042 (correct as uint32)
```

- **Addition is simpler**: You can start adding from the lowest address (where the LSB lives), which is the natural direction for addition with carry propagation.

### Big-Endian Advantages

- **Human-readable**: Hex dumps read in the same order you write numbers. `0x12345678` appears as `12 34 56 78` in memory.
- **String/number comparison**: Byte-by-byte comparison from the first byte works for both strings and numbers (the most significant byte is first for both).
- **Network standard**: Being the network byte order, big-endian data needs no conversion in network code on big-endian machines.

---

## Detecting and Converting Endianness

### C++

```cpp
#include <iostream>
#include <cstdint>
#include <cstring>
#include <bit>        // C++20: std::endian
#include <algorithm>  // std::reverse

// Method 1: Detect endianness at runtime
bool isLittleEndian() {
    uint32_t value = 1;      // 0x00000001
    uint8_t* bytes = reinterpret_cast<uint8_t*>(&value);
    return bytes[0] == 1;    // If LSB is at lowest address -> little-endian
}

// Method 2: C++20 std::endian (compile-time)
void detectEndianCpp20() {
    if constexpr (std::endian::native == std::endian::little) {
        std::cout << "Little-endian" << std::endl;
    } else if constexpr (std::endian::native == std::endian::big) {
        std::cout << "Big-endian" << std::endl;
    } else {
        std::cout << "Mixed endianness" << std::endl;
    }
}

// Byte swap functions
uint16_t swapBytes16(uint16_t value) {
    return (value >> 8) | (value << 8);
}

uint32_t swapBytes32(uint32_t value) {
    return ((value >> 24) & 0x000000FF) |
           ((value >>  8) & 0x0000FF00) |
           ((value <<  8) & 0x00FF0000) |
           ((value << 24) & 0xFF000000);
}

// C++23: std::byteswap
// uint32_t swapped = std::byteswap(value);

int main() {
    std::cout << "System is " << (isLittleEndian() ? "little" : "big")
              << "-endian" << std::endl;

    detectEndianCpp20();

    // Byte swap example
    uint32_t host_value = 0x12345678;
    uint32_t swapped = swapBytes32(host_value);
    std::cout << std::hex;
    std::cout << "Original: 0x" << host_value << std::endl;  // 0x12345678
    std::cout << "Swapped:  0x" << swapped    << std::endl;  // 0x78563412

    // Network byte order conversion (POSIX)
    // #include <arpa/inet.h>  // Linux/macOS
    // #include <winsock2.h>   // Windows
    // uint32_t net = htonl(host_value);   // host to network long
    // uint16_t net16 = htons(port);       // host to network short
    // uint32_t host = ntohl(net);         // network to host long
    // uint16_t host16 = ntohs(net16);     // network to host short

    return 0;
}
```

```ad-tip
title: Compiler intrinsics
Most compilers offer optimized byte-swap intrinsics:
- GCC/Clang: `__builtin_bswap16()`, `__builtin_bswap32()`, `__builtin_bswap64()`
- MSVC: `_byteswap_ushort()`, `_byteswap_ulong()`, `_byteswap_uint64()`

These compile to single instructions on architectures that support them (e.g., `BSWAP` on x86).
```

### C#

```csharp
using System;
using System.Buffers.Binary;  // .NET Core 2.1+ / .NET 5+

class Program
{
    static void Main()
    {
        // Detect endianness
        Console.WriteLine($"Is little-endian: {BitConverter.IsLittleEndian}");  // True on x86/x64

        // View bytes of an integer
        int value = 0x12345678;
        byte[] bytes = BitConverter.GetBytes(value);
        Console.WriteLine("Bytes in memory: " +
            BitConverter.ToString(bytes));
        // Little-endian: 78-56-34-12
        // Big-endian:    12-34-56-78

        // Manual byte swap
        int swapped = System.Net.IPAddress.HostToNetworkOrder(value);
        Console.WriteLine($"Host order:    0x{value:X8}");    // 0x12345678
        Console.WriteLine($"Network order: 0x{swapped:X8}");  // 0x78563412

        // BinaryPrimitives (modern .NET - preferred approach)
        byte[] buffer = new byte[4];
        BinaryPrimitives.WriteInt32BigEndian(buffer, value);
        Console.WriteLine("Big-endian bytes:    " + BitConverter.ToString(buffer));
        // 12-34-56-78

        BinaryPrimitives.WriteInt32LittleEndian(buffer, value);
        Console.WriteLine("Little-endian bytes: " + BitConverter.ToString(buffer));
        // 78-56-34-12

        // Read back with explicit endianness
        int fromBE = BinaryPrimitives.ReadInt32BigEndian(new byte[] { 0x12, 0x34, 0x56, 0x78 });
        int fromLE = BinaryPrimitives.ReadInt32LittleEndian(new byte[] { 0x78, 0x56, 0x34, 0x12 });
        Console.WriteLine($"From BE: 0x{fromBE:X8}");  // 0x12345678
        Console.WriteLine($"From LE: 0x{fromLE:X8}");  // 0x12345678

        // Reverse bytes manually using Span
        Span<byte> span = stackalloc byte[] { 0x12, 0x34, 0x56, 0x78 };
        span.Reverse();
        Console.WriteLine("Reversed: " + BitConverter.ToString(span.ToArray()));
        // 78-56-34-12
    }
}
```

```ad-tip
title: BinaryPrimitives vs BitConverter
`BinaryPrimitives` (in `System.Buffers.Binary`) is the modern, allocation-free way to read/write with explicit endianness in .NET. Prefer it over `BitConverter` + `Array.Reverse()` for performance-sensitive code. See [[Bit Manipulation in CSharp]] for more .NET-specific patterns.
```

### JavaScript

```javascript
// Detect endianness
function getEndianness() {
    let buffer = new ArrayBuffer(2);
    let uint16 = new Uint16Array(buffer);
    let uint8  = new Uint8Array(buffer);

    uint16[0] = 0x0102;

    if (uint8[0] === 0x01 && uint8[1] === 0x02) {
        return 'big-endian';
    } else if (uint8[0] === 0x02 && uint8[1] === 0x01) {
        return 'little-endian';
    }
    return 'unknown';
}

console.log(`System endianness: ${getEndianness()}`);  // Usually "little-endian"

// DataView: read/write with explicit endianness
let buffer = new ArrayBuffer(4);
let view = new DataView(buffer);

// Write 0x12345678 in big-endian
view.setUint32(0, 0x12345678, false);  // false = big-endian
console.log('Big-endian bytes:');
let bytes = new Uint8Array(buffer);
console.log([...bytes].map(b => '0x' + b.toString(16).padStart(2, '0')).join(' '));
// 0x12 0x34 0x56 0x78

// Write 0x12345678 in little-endian
view.setUint32(0, 0x12345678, true);   // true = little-endian
console.log('Little-endian bytes:');
bytes = new Uint8Array(buffer);
console.log([...bytes].map(b => '0x' + b.toString(16).padStart(2, '0')).join(' '));
// 0x78 0x56 0x34 0x12

// Read with explicit endianness
view.setUint32(0, 0x12345678, true);  // stored as little-endian
let readBE = view.getUint32(0, false); // read as big-endian
let readLE = view.getUint32(0, true);  // read as little-endian
console.log(`Read as BE: 0x${readBE.toString(16)}`);  // 0x78563412 (wrong!)
console.log(`Read as LE: 0x${readLE.toString(16)}`);  // 0x12345678 (correct)

// Manual byte swap (32-bit)
function swapBytes32(val) {
    return ((val & 0xFF000000) >>> 24) |
           ((val & 0x00FF0000) >>> 8)  |
           ((val & 0x0000FF00) << 8)   |
           ((val & 0x000000FF) << 24);
}

console.log(`Swapped: 0x${(swapBytes32(0x12345678) >>> 0).toString(16)}`);
// 0x78563412
```

```ad-warning
title: TypedArrays use platform endianness
`Uint16Array`, `Int32Array`, `Float64Array`, etc. use the **platform's native endianness** (almost always little-endian in browsers). For cross-platform binary data, always use `DataView` with explicit endianness. See [[Bit Manipulation in JavaScript]] for more details.
```

---

## Endianness in Network Programming

Network byte order is **big-endian**. The standard conversion functions are:

| Function   | Direction            | Description                    |
|-----------|---------------------|--------------------------------|
| `htons()` | Host to Network, Short | Convert 16-bit to big-endian  |
| `htonl()` | Host to Network, Long  | Convert 32-bit to big-endian  |
| `ntohs()` | Network to Host, Short | Convert 16-bit from big-endian|
| `ntohl()` | Network to Host, Long  | Convert 32-bit from big-endian|

```
Sending port 8080 (0x1F90) over the network:

Little-endian host memory:  | 0x90 | 0x1F |

After htons():              | 0x1F | 0x90 |  (big-endian, ready to send)

Receiving side (also little-endian):
Received:                   | 0x1F | 0x90 |
After ntohs():              | 0x90 | 0x1F |  (back to host order = 8080)
```

See [[Networking and Subnet Masks]] for more on how endianness affects network programming.

---

## Common Pitfalls

### 1. Casting Pointers to Different-Sized Types

```
uint32_t x = 0x12345678;
uint8_t* p = (uint8_t*)&x;

On little-endian:   p[0] = 0x78, p[1] = 0x56, p[2] = 0x34, p[3] = 0x12
On big-endian:      p[0] = 0x12, p[1] = 0x34, p[2] = 0x56, p[3] = 0x78
```

### 2. Reading Binary Files from Other Platforms

Always check the file format specification for endianness. Many formats include a **byte order mark (BOM)** or **magic number** to identify endianness.

### 3. Sending Structs Over the Network

Never send raw structs over a network — different machines may have different endianness AND different struct padding/alignment. Use serialization with explicit byte ordering instead.

```ad-warning
title: Struct padding + endianness = double trouble
Even if two machines have the same endianness, struct layout can differ due to compiler-specific padding and alignment rules. Always serialize field by field with explicit byte order. See [[Bits Bytes and Words]] for more on alignment and padding.
```

---

## Related Notes

- [[Bits Bytes and Words]] — data sizes, memory layout, alignment
- [[Networking and Subnet Masks]] — network byte order in practice
- [[Bit Manipulation in CSharp]] — .NET's BinaryPrimitives and BitConverter
- [[Bit Manipulation in CPP]] — C++ byte-swapping techniques
- [[Bit Manipulation in JavaScript]] — DataView and TypedArrays
- [[Binary Number System]] — foundational binary concepts
