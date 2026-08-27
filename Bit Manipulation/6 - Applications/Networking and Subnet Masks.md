---
title: Networking and Subnet Masks
tags:
  - bit-manipulation
  - application
  - networking
  - subnet
aliases:
  - Subnet Masks
  - IP Address Bit Manipulation
  - CIDR Notation
date: 2026-08-19
---

## Networking and Subnet Masks

IP networking is one of the most elegant real-world applications of [[Binary Number System|binary]] and bitwise operations. Every IPv4 address is a 32-bit integer, every subnet mask is a contiguous run of 1-bits followed by 0-bits, and every routing decision boils down to a single [[AND Operator|AND]] operation. Understanding how addresses, masks, and subnets work at the bit level turns networking from memorized rules into something you can derive from first principles.

This note covers how IPv4 addresses map to 32-bit integers, how subnet masks partition those bits into network and host portions, how CIDR notation encodes mask length, and the bitwise formulas for computing network addresses, broadcast addresses, host ranges, and same-subnet checks. Complete code examples are provided in C#, C++, and JavaScript.

---

## IPv4 Addresses as 32-Bit Integers

An IPv4 address is written as four decimal numbers separated by dots (e.g., `192.168.1.1`), but under the hood it is a single **32-bit unsigned integer**. Each of the four numbers is one **octet** -- an 8-bit value ranging from 0 to 255 (see [[Bits Bytes and Words]]).

### Dotted-Decimal to Binary

The address `192.168.1.1` breaks down as:

| Octet    | Decimal | Hexadecimal | Binary     |
|:--------:|:-------:|:-----------:|:----------:|
| 1st      | 192     | `0xC0`      | `11000000` |
| 2nd      | 168     | `0xA8`      | `10101000` |
| 3rd      | 1       | `0x01`      | `00000001` |
| 4th      | 1       | `0x01`      | `00000001` |

Packed into a single 32-bit value using [[Left Shift]] and [[OR Operator]]:

```
  192 << 24 = 11000000  00000000  00000000  00000000
  168 << 16 = 00000000  10101000  00000000  00000000
    1 << 8  = 00000000  00000000  00000001  00000000
    1       = 00000000  00000000  00000000  00000001
              ─────────────────────────────────────────
  OR all    = 11000000  10101000  00000001  00000001
            = 0xC0A80101
            = 3,232,235,777 (decimal)
```

**Formula:** `ip = (octet1 << 24) | (octet2 << 16) | (octet3 << 8) | octet4`

This is the same packing technique used for [[Color and Graphics|ARGB colors]] -- four bytes merged into one integer with shifts and OR.

### Extracting Octets

To extract individual octets from a packed IP, use [[Right Shift]] and [[AND Operator]] with a `0xFF` mask (see [[Creating Bit Masks]]):

| Octet | Formula                | Shift |
|:-----:|------------------------|:-----:|
| 1st   | `(ip >> 24) & 0xFF`   | 24    |
| 2nd   | `(ip >> 16) & 0xFF`   | 16    |
| 3rd   | `(ip >> 8) & 0xFF`    | 8     |
| 4th   | `ip & 0xFF`           | 0     |

```ad-note
title: Network Byte Order (Big-Endian)
Network protocols transmit the most significant byte first ([[Endianness|big-endian]]). The first octet in the dotted-decimal string occupies the highest byte (bits 31-24) of the integer. This is called **network byte order**. On little-endian machines (x86, ARM), you must convert between host and network byte order using functions like `htonl()` / `ntohl()` in C/C++ or `IPAddress.HostToNetworkOrder()` in C#.
```

---

## Subnet Masks

A subnet mask is a 32-bit value that divides an IP address into two parts:

- **Network portion** -- the leading bits that identify the network (all 1s in the mask)
- **Host portion** -- the trailing bits that identify a specific device on that network (all 0s in the mask)

### The Contiguous 1s Rule

A valid subnet mask is always a contiguous block of 1-bits followed by a contiguous block of 0-bits. There are never "gaps" -- you will never see a valid mask like `11110011...`. This contiguity is what makes masks work cleanly with a single AND operation.

Common subnet masks:

| Mask (Dotted)     | Mask (Hex)     | Binary                                  | Prefix |
|:-----------------:|:--------------:|:---------------------------------------:|:------:|
| `255.0.0.0`       | `0xFF000000`   | `11111111 00000000 00000000 00000000`   | /8     |
| `255.255.0.0`     | `0xFFFF0000`   | `11111111 11111111 00000000 00000000`   | /16    |
| `255.255.255.0`   | `0xFFFFFF00`   | `11111111 11111111 11111111 00000000`   | /24    |
| `255.255.255.128` | `0xFFFFFF80`   | `11111111 11111111 11111111 10000000`   | /25    |
| `255.255.255.192` | `0xFFFFFFC0`   | `11111111 11111111 11111111 11000000`   | /26    |
| `255.255.255.240` | `0xFFFFFFF0`   | `11111111 11111111 11111111 11110000`   | /28    |

```ad-tip
title: Quick Mask Values for the Last Octet
The valid last-octet values for subnet masks follow a pattern: 0, 128, 192, 224, 240, 248, 252, 254, 255. Each is `256 - 2^n` for n = 8, 7, 6, ..., 0. Memorize these and you can instantly decode any subnet mask.
```

### ASCII Art: Subnet Mask Applied to an IP

```
IP Address:   192.168.1.100
Subnet Mask:  255.255.255.0   (/24)

Bit Position: 31              23              15              7             0
              |               |               |               |             |
IP (binary):  11000000 . 10101000 . 00000001 . 01100100
Mask:         11111111 . 11111111 . 11111111 . 00000000
              ├────────────────────────────────┤├──────┤
              │      Network Portion (24 bits) ││ Host │
              │      Identified by the 1s      ││ (8b) │
              └────────────────────────────────┘└──────┘

ip & mask:    11000000 . 10101000 . 00000001 . 00000000  = 192.168.1.0   (Network)
ip & ~mask:   00000000 . 00000000 . 00000000 . 01100100  = 0.0.0.100     (Host)
ip | ~mask:   11000000 . 10101000 . 00000001 . 11111111  = 192.168.1.255 (Broadcast)
```

---

## CIDR Notation

**Classless Inter-Domain Routing (CIDR)** notation expresses the subnet mask as a single number: the count of leading 1-bits. For example, `/24` means 24 leading 1-bits followed by 8 trailing 0-bits.

### Computing the Mask from a Prefix Length

Given a prefix length `n`, compute the 32-bit mask:

**Formula:** `mask = 0xFFFFFFFF << (32 - n)`

Or equivalently: `mask = ~((1 << (32 - n)) - 1)`

Step-by-step for `/24`:

```
Step 1: 32 - 24 = 8 (number of host bits)

Method 1: Left-shift all-ones
    0xFFFFFFFF << 8
    = 11111111 11111111 11111111 11111111 << 8
    = 11111111 11111111 11111111 00000000
    = 0xFFFFFF00

Method 2: Complement approach
    1 << 8    = 00000000 00000000 00000001 00000000  (256)
    256 - 1   = 00000000 00000000 00000000 11111111  (255)
    ~255      = 11111111 11111111 11111111 00000000
    = 0xFFFFFF00
```

Both approaches produce the same result. Method 1 is more common in practice.

```ad-warning
title: Shift Width and Integer Size
When computing masks with `0xFFFFFFFF << (32 - n)`, be careful with integer widths. In C#, `0xFFFFFFFF` is a `uint` literal. In C++, use `0xFFFFFFFFU` or `uint32_t`. In JavaScript, `0xFFFFFFFF << 0` produces `-1` (signed), so use `>>> 0` to get the unsigned value. When `n = 0`, you would shift by 32 -- this is undefined behavior in C/C++ and produces 0 in JavaScript. Handle `/0` as a special case.
```

### CIDR Reference Table

| CIDR  | Mask               | Host Bits | Total Addresses | Usable Hosts |
|:-----:|:------------------:|:---------:|:---------------:|:------------:|
| /8    | 255.0.0.0          | 24        | 16,777,216      | 16,777,214   |
| /16   | 255.255.0.0        | 16        | 65,536          | 65,534       |
| /20   | 255.255.240.0      | 12        | 4,096           | 4,094        |
| /24   | 255.255.255.0      | 8         | 256             | 254          |
| /25   | 255.255.255.128    | 7         | 128             | 126          |
| /26   | 255.255.255.192    | 6         | 64              | 62           |
| /27   | 255.255.255.224    | 5         | 32              | 30           |
| /28   | 255.255.255.240    | 4         | 16              | 14           |
| /29   | 255.255.255.248    | 3         | 8               | 6            |
| /30   | 255.255.255.252    | 2         | 4               | 2            |
| /31   | 255.255.255.254    | 1         | 2               | 2*           |
| /32   | 255.255.255.255    | 0         | 1               | 1*           |

*`/31` subnets (RFC 3021) are used for point-to-point links with no network/broadcast address. `/32` identifies a single host.*

---

## Core Bitwise Operations

Every subnet calculation reduces to a handful of bitwise operations using [[AND Operator]], [[OR Operator]], and [[NOT Operator]].

### Network Address: `ip & mask`

The network address is the first address in a subnet. It identifies the network itself. Computed by AND-ing the IP with the mask, which zeroes out the host bits:

```
IP:           11000000 10101000 00000001 01100100   (192.168.1.100)
Mask:         11111111 11111111 11111111 00000000   (255.255.255.0)
              ──────────────────────────────────────
ip & mask:    11000000 10101000 00000001 00000000   (192.168.1.0)
```

The [[AND Operator]] preserves the network bits (where mask = 1) and clears the host bits (where mask = 0). This is exactly the same "mask and extract" pattern used in [[Combining and Extracting with Masks]].

### Broadcast Address: `ip | ~mask`

The broadcast address is the last address in a subnet. Packets sent to this address reach all hosts on the network. Computed by OR-ing the IP with the complement of the mask, which sets all host bits to 1:

```
Mask:         11111111 11111111 11111111 00000000   (255.255.255.0)
~Mask:        00000000 00000000 00000000 11111111   (0.0.0.255)

IP:           11000000 10101000 00000001 01100100   (192.168.1.100)
~Mask:        00000000 00000000 00000000 11111111   (0.0.0.255)
              ──────────────────────────────────────
ip | ~mask:   11000000 10101000 00000001 11111111   (192.168.1.255)
```

The [[OR Operator]] keeps the network bits intact (OR with 0 preserves) and forces all host bits to 1 (OR with 1 sets).

### Host Part: `ip & ~mask`

The host part isolates just the device identifier within the subnet:

```
IP:           11000000 10101000 00000001 01100100   (192.168.1.100)
~Mask:        00000000 00000000 00000000 11111111   (0.0.0.255)
              ──────────────────────────────────────
ip & ~mask:   00000000 00000000 00000000 01100100   (0.0.0.100 = host 100)
```

### Number of Usable Hosts

The number of host addresses in a subnet is determined by the host bits:

**Formula:** `totalAddresses = 2^(32 - prefix)`

**Formula:** `usableHosts = 2^(32 - prefix) - 2`

The `-2` subtracts the network address (all host bits = 0) and the broadcast address (all host bits = 1), neither of which can be assigned to a device.

Using bit manipulation: `totalAddresses = 1 << (32 - prefix)` (see [[Left Shift]])

```ad-warning
title: Edge Cases: /31 and /32
For `/31` subnets (RFC 3021), used on point-to-point links, both addresses are usable -- there is no separate network or broadcast address. For `/32`, the "subnet" contains exactly one address and is used to identify a single host (e.g., in routing tables or firewall rules). The `2^n - 2` formula does not apply to these special cases.
```

### Same-Subnet Check: `(ip1 & mask) == (ip2 & mask)`

Two IP addresses are on the same subnet if and only if their network portions are identical. Mask both addresses and compare:

```
IP1:           11000000 10101000 00000001 01100100   (192.168.1.100)
IP2:           11000000 10101000 00000001 11001000   (192.168.1.200)
Mask:          11111111 11111111 11111111 00000000   (255.255.255.0)

IP1 & Mask:    11000000 10101000 00000001 00000000   (192.168.1.0)
IP2 & Mask:    11000000 10101000 00000001 00000000   (192.168.1.0)

Same network? YES (both produce 192.168.1.0)

IP3:           10101100 00010000 00000010 00001010   (172.16.2.10)
IP3 & Mask:    10101100 00010000 00000010 00000000   (172.16.2.0)

Same as IP1?   NO (192.168.1.0 != 172.16.2.0)
```

This single AND + compare is how routers make forwarding decisions millions of times per second.

```ad-tip
title: Routing Tables Are Just Mask-and-Compare
Every entry in a router's forwarding table is a (network address, mask) pair. For each incoming packet, the router AND-s the destination IP with each mask and checks if it matches the network address. The longest matching prefix (most specific mask) wins. This is the core of IP routing, and it is fundamentally a bitwise AND comparison.
```

---

## Code Examples

The following examples implement a complete subnet calculator in C#, C++, and JavaScript. Each version can parse IP strings to 32-bit integers, apply subnet masks, compute network/broadcast/host-range, and check same-subnet membership.

### C#

```csharp
using System;

public static class SubnetCalculator
{
    /// <summary>Parse dotted-decimal IP string to 32-bit unsigned integer.</summary>
    public static uint ParseIP(string ip)
    {
        string[] octets = ip.Split('.');
        if (octets.Length != 4)
            throw new ArgumentException($"Invalid IP: {ip}");

        uint result = 0;
        for (int i = 0; i < 4; i++)
        {
            uint octet = uint.Parse(octets[i]);
            if (octet > 255)
                throw new ArgumentException($"Octet out of range: {octet}");
            result |= octet << (24 - i * 8);
        }
        return result;
    }

    /// <summary>Convert 32-bit integer back to dotted-decimal string.</summary>
    public static string ToIPString(uint ip)
    {
        return $"{(ip >> 24) & 0xFF}.{(ip >> 16) & 0xFF}" +
               $".{(ip >> 8) & 0xFF}.{ip & 0xFF}";
    }

    /// <summary>Compute subnet mask from CIDR prefix length (0-32).</summary>
    public static uint CidrToMask(int prefix)
    {
        if (prefix < 0 || prefix > 32)
            throw new ArgumentException($"Invalid prefix: {prefix}");
        if (prefix == 0) return 0;
        return 0xFFFFFFFF << (32 - prefix);
    }

    /// <summary>Network address = ip & mask.</summary>
    public static uint NetworkAddress(uint ip, uint mask)
    {
        return ip & mask;
    }

    /// <summary>Broadcast address = ip | ~mask.</summary>
    public static uint BroadcastAddress(uint ip, uint mask)
    {
        return ip | ~mask;
    }

    /// <summary>Host part = ip & ~mask.</summary>
    public static uint HostPart(uint ip, uint mask)
    {
        return ip & ~mask;
    }

    /// <summary>Number of usable host addresses.</summary>
    public static uint UsableHosts(int prefix)
    {
        if (prefix >= 31) return (uint)(prefix == 32 ? 1 : 2);
        return (1u << (32 - prefix)) - 2;
    }

    /// <summary>First usable host = network + 1.</summary>
    public static uint FirstHost(uint ip, uint mask)
    {
        return NetworkAddress(ip, mask) + 1;
    }

    /// <summary>Last usable host = broadcast - 1.</summary>
    public static uint LastHost(uint ip, uint mask)
    {
        return BroadcastAddress(ip, mask) - 1;
    }

    /// <summary>Check if two IPs are on the same subnet.</summary>
    public static bool SameSubnet(uint ip1, uint ip2, uint mask)
    {
        return (ip1 & mask) == (ip2 & mask);
    }
}

class Program
{
    static void Main()
    {
        uint ip   = SubnetCalculator.ParseIP("192.168.1.100");
        int prefix = 24;
        uint mask = SubnetCalculator.CidrToMask(prefix);

        Console.WriteLine($"IP Address:        {SubnetCalculator.ToIPString(ip)}");
        Console.WriteLine($"Subnet Mask:       {SubnetCalculator.ToIPString(mask)} (/{prefix})");
        Console.WriteLine($"Network Address:   {SubnetCalculator.ToIPString(SubnetCalculator.NetworkAddress(ip, mask))}");
        Console.WriteLine($"Broadcast Address: {SubnetCalculator.ToIPString(SubnetCalculator.BroadcastAddress(ip, mask))}");
        Console.WriteLine($"Host Part:         {SubnetCalculator.ToIPString(SubnetCalculator.HostPart(ip, mask))}");
        Console.WriteLine($"First Usable Host: {SubnetCalculator.ToIPString(SubnetCalculator.FirstHost(ip, mask))}");
        Console.WriteLine($"Last Usable Host:  {SubnetCalculator.ToIPString(SubnetCalculator.LastHost(ip, mask))}");
        Console.WriteLine($"Usable Hosts:      {SubnetCalculator.UsableHosts(prefix)}");
        Console.WriteLine($"Hex:               0x{ip:X8}");
        // Output:
        // IP Address:        192.168.1.100
        // Subnet Mask:       255.255.255.0 (/24)
        // Network Address:   192.168.1.0
        // Broadcast Address: 192.168.1.255
        // Host Part:         0.0.0.100
        // First Usable Host: 192.168.1.1
        // Last Usable Host:  192.168.1.254
        // Usable Hosts:      254
        // Hex:               0xC0A80164

        // Same-subnet check
        uint ip2 = SubnetCalculator.ParseIP("192.168.1.200");
        uint ip3 = SubnetCalculator.ParseIP("192.168.2.100");
        Console.WriteLine($"\n192.168.1.100 and 192.168.1.200 same /24? " +
            $"{SubnetCalculator.SameSubnet(ip, ip2, mask)}");   // True
        Console.WriteLine($"192.168.1.100 and 192.168.2.100 same /24? " +
            $"{SubnetCalculator.SameSubnet(ip, ip3, mask)}");   // False
    }
}
```

```ad-note
title: uint vs int in C#
Subnet masks with the top bit set (any prefix /1 or shorter, or masks like 255.x.x.x) will be negative if stored in a signed `int`. Always use `uint` for IP addresses and masks to avoid sign-related bugs. The `~` operator on a `uint` correctly produces the unsigned complement.
```

### C++

```cpp
#include <iostream>
#include <string>
#include <sstream>
#include <cstdint>
#include <stdexcept>

// Parse dotted-decimal IP string to 32-bit unsigned integer
uint32_t parseIP(const std::string& ip) {
    uint32_t result = 0;
    uint32_t octet = 0;
    int octetCount = 0;
    int shift = 24;

    for (size_t i = 0; i <= ip.size(); ++i) {
        if (i == ip.size() || ip[i] == '.') {
            if (octet > 255)
                throw std::invalid_argument("Octet out of range");
            result |= (octet << shift);
            shift -= 8;
            octet = 0;
            ++octetCount;
        } else if (ip[i] >= '0' && ip[i] <= '9') {
            octet = octet * 10 + (ip[i] - '0');
        } else {
            throw std::invalid_argument("Invalid character in IP");
        }
    }

    if (octetCount != 4)
        throw std::invalid_argument("Invalid IP format");
    return result;
}

// Convert 32-bit integer to dotted-decimal string
std::string toIPString(uint32_t ip) {
    return std::to_string((ip >> 24) & 0xFF) + "." +
           std::to_string((ip >> 16) & 0xFF) + "." +
           std::to_string((ip >> 8)  & 0xFF) + "." +
           std::to_string(ip & 0xFF);
}

// Compute subnet mask from CIDR prefix length
uint32_t cidrToMask(int prefix) {
    if (prefix < 0 || prefix > 32)
        throw std::invalid_argument("Invalid prefix length");
    if (prefix == 0) return 0;
    return 0xFFFFFFFFU << (32 - prefix);
}

// Network address = ip & mask
uint32_t networkAddress(uint32_t ip, uint32_t mask) {
    return ip & mask;
}

// Broadcast address = ip | ~mask
uint32_t broadcastAddress(uint32_t ip, uint32_t mask) {
    return ip | ~mask;
}

// Host part = ip & ~mask
uint32_t hostPart(uint32_t ip, uint32_t mask) {
    return ip & ~mask;
}

// Number of usable hosts
uint32_t usableHosts(int prefix) {
    if (prefix >= 31) return (prefix == 32) ? 1 : 2;
    return (1U << (32 - prefix)) - 2;
}

// First usable host = network + 1
uint32_t firstHost(uint32_t ip, uint32_t mask) {
    return networkAddress(ip, mask) + 1;
}

// Last usable host = broadcast - 1
uint32_t lastHost(uint32_t ip, uint32_t mask) {
    return broadcastAddress(ip, mask) - 1;
}

// Check if two IPs are on the same subnet
bool sameSubnet(uint32_t ip1, uint32_t ip2, uint32_t mask) {
    return (ip1 & mask) == (ip2 & mask);
}

int main() {
    uint32_t ip = parseIP("192.168.1.100");
    int prefix  = 24;
    uint32_t mask = cidrToMask(prefix);

    std::cout << "IP Address:        " << toIPString(ip) << "\n";
    std::cout << "Subnet Mask:       " << toIPString(mask)
              << " (/" << prefix << ")\n";
    std::cout << "Network Address:   " << toIPString(networkAddress(ip, mask)) << "\n";
    std::cout << "Broadcast Address: " << toIPString(broadcastAddress(ip, mask)) << "\n";
    std::cout << "Host Part:         " << toIPString(hostPart(ip, mask)) << "\n";
    std::cout << "First Usable Host: " << toIPString(firstHost(ip, mask)) << "\n";
    std::cout << "Last Usable Host:  " << toIPString(lastHost(ip, mask)) << "\n";
    std::cout << "Usable Hosts:      " << usableHosts(prefix) << "\n";

    // Same-subnet check
    uint32_t ip2 = parseIP("192.168.1.200");
    uint32_t ip3 = parseIP("192.168.2.100");

    std::cout << "\n192.168.1.100 and 192.168.1.200 same /24? "
              << std::boolalpha << sameSubnet(ip, ip2, mask) << "\n";   // true
    std::cout << "192.168.1.100 and 192.168.2.100 same /24? "
              << sameSubnet(ip, ip3, mask) << "\n";                     // false

    return 0;
}
```

```ad-warning
title: Use Unsigned Types in C++
Always use `uint32_t` (from `<cstdint>`) for IP addresses and masks. Right-shifting a signed negative integer is **implementation-defined** in C++, and left-shifting into or past the sign bit is **undefined behavior**. Using unsigned types guarantees logical (zero-filling) shifts and well-defined behavior for all mask computations. The `0xFFFFFFFFU` suffix ensures the literal is unsigned.
```

### JavaScript

```javascript
// Parse dotted-decimal IP string to 32-bit unsigned integer
function parseIP(ip) {
    const parts = ip.split(".");
    if (parts.length !== 4)
        throw new Error(`Invalid IP: ${ip}`);

    let result = 0;
    for (let i = 0; i < 4; i++) {
        const octet = parseInt(parts[i], 10);
        if (isNaN(octet) || octet < 0 || octet > 255)
            throw new Error(`Octet out of range: ${parts[i]}`);
        result = ((result << 8) | octet) >>> 0;  // >>> 0 keeps unsigned
    }
    return result;
}

// Convert 32-bit integer to dotted-decimal string
function toIPString(ip) {
    return `${(ip >>> 24) & 0xFF}.${(ip >>> 16) & 0xFF}` +
           `.${(ip >>> 8) & 0xFF}.${ip & 0xFF}`;
}

// Compute subnet mask from CIDR prefix length
function cidrToMask(prefix) {
    if (prefix < 0 || prefix > 32)
        throw new Error(`Invalid prefix: ${prefix}`);
    if (prefix === 0) return 0;
    // (-1 << (32 - prefix)) >>> 0 converts to unsigned
    return (0xFFFFFFFF << (32 - prefix)) >>> 0;
}

// Network address = ip & mask
function networkAddress(ip, mask) {
    return (ip & mask) >>> 0;
}

// Broadcast address = ip | ~mask
function broadcastAddress(ip, mask) {
    return (ip | (~mask >>> 0)) >>> 0;
}

// Host part = ip & ~mask
function hostPart(ip, mask) {
    return (ip & (~mask >>> 0)) >>> 0;
}

// Number of usable hosts
function usableHosts(prefix) {
    if (prefix >= 31) return prefix === 32 ? 1 : 2;
    return (1 << (32 - prefix)) - 2;
}

// First usable host = network + 1
function firstHost(ip, mask) {
    return (networkAddress(ip, mask) + 1) >>> 0;
}

// Last usable host = broadcast - 1
function lastHost(ip, mask) {
    return (broadcastAddress(ip, mask) - 1) >>> 0;
}

// Check if two IPs are on the same subnet
function sameSubnet(ip1, ip2, mask) {
    return (ip1 & mask) === (ip2 & mask);
}

// ===== Usage =====
const ip     = parseIP("192.168.1.100");
const prefix = 24;
const mask   = cidrToMask(prefix);

console.log(`IP Address:        ${toIPString(ip)}`);
console.log(`Subnet Mask:       ${toIPString(mask)} (/${prefix})`);
console.log(`Network Address:   ${toIPString(networkAddress(ip, mask))}`);
console.log(`Broadcast Address: ${toIPString(broadcastAddress(ip, mask))}`);
console.log(`Host Part:         ${toIPString(hostPart(ip, mask))}`);
console.log(`First Usable Host: ${toIPString(firstHost(ip, mask))}`);
console.log(`Last Usable Host:  ${toIPString(lastHost(ip, mask))}`);
console.log(`Usable Hosts:      ${usableHosts(prefix)}`);
// Output:
// IP Address:        192.168.1.100
// Subnet Mask:       255.255.255.0 (/24)
// Network Address:   192.168.1.0
// Broadcast Address: 192.168.1.255
// Host Part:         0.0.0.100
// First Usable Host: 192.168.1.1
// Last Usable Host:  192.168.1.254
// Usable Hosts:      254

// Same-subnet check
const ip2 = parseIP("192.168.1.200");
const ip3 = parseIP("192.168.2.100");

console.log(`\n192.168.1.100 and 192.168.1.200 same /24? ${sameSubnet(ip, ip2, mask)}`);
console.log(`192.168.1.100 and 192.168.2.100 same /24? ${sameSubnet(ip, ip3, mask)}`);
```

```ad-warning
title: JavaScript's >>> 0 Is Essential
JavaScript bitwise operators produce signed 32-bit results. Any IP address or mask with bit 31 set (e.g., addresses starting with 128-255, or any mask /1 or longer) will appear as a negative number. The `>>> 0` (unsigned right shift by zero) trick converts to unsigned 32-bit. Without it, `parseIP("192.168.1.1")` returns `-1062731519` instead of `3232235777`. Always apply `>>> 0` after operations that may set bit 31. See [[Unsigned Right Shift]] for details.
```

---

## Bitwise Operations Quick Reference for Subnets

| Operation                    | Formula                                  | Operators Used                                    |
|------------------------------|------------------------------------------|---------------------------------------------------|
| Parse IP to integer          | `(o1 << 24) \| (o2 << 16) \| (o3 << 8) \| o4` | [[Left Shift]], [[OR Operator]]             |
| Extract octet N (0=highest)  | `(ip >> (24 - N*8)) & 0xFF`             | [[Right Shift]], [[AND Operator]]                 |
| Mask from prefix             | `0xFFFFFFFF << (32 - prefix)`           | [[Left Shift]]                                    |
| Network address              | `ip & mask`                              | [[AND Operator]]                                  |
| Broadcast address            | `ip \| ~mask`                            | [[OR Operator]], [[NOT Operator]]                 |
| Host part                    | `ip & ~mask`                             | [[AND Operator]], [[NOT Operator]]                |
| Total addresses              | `1 << (32 - prefix)`                     | [[Left Shift]]                                    |
| Same subnet                  | `(ip1 & mask) == (ip2 & mask)`           | [[AND Operator]]                                  |
| Wildcard mask                | `~mask`                                  | [[NOT Operator]]                                  |

```ad-tip
title: Wildcard Masks
Some systems (notably Cisco ACLs and OSPF) use **wildcard masks** instead of subnet masks. A wildcard mask is simply the bitwise complement of the subnet mask (`~mask`). Where a subnet mask has 1s (match these bits), a wildcard mask has 0s, and vice versa. For `/24`: subnet mask = `255.255.255.0`, wildcard = `0.0.0.255`. The relationship is always `wildcard = ~subnet` and `subnet = ~wildcard`.
```

---

## Common Pitfalls and Best Practices

### Pitfalls to Avoid

1. **Forgetting unsigned types** -- IP addresses with first octets >= 128 set bit 31. In signed integers this is negative, breaking comparisons and shifts. Always use `uint` (C#), `uint32_t` (C++), or `>>> 0` (JavaScript).

2. **Shifting by 32** -- When prefix = 0, the formula `0xFFFFFFFF << 32` is undefined behavior in C/C++ and produces unexpected results in JavaScript. Always handle prefix 0 as a special case returning `0`.

3. **Confusing network and host byte order** -- The integer representation in your code may not match what the network sends. Use `htonl()`/`ntohl()` in C/C++ when interfacing with sockets (see [[Endianness]]).

4. **Applying the `-2` formula to /31 and /32** -- These special prefix lengths do not follow the standard `2^n - 2` rule. Handle them explicitly.

5. **Invalid subnet masks** -- A mask with non-contiguous 1-bits (e.g., `255.0.255.0`) is not valid. If accepting user input, validate that the mask is contiguous: `mask & (~mask + 1)` should equal `~mask + 1` (the lowest set bit of `~mask` should be contiguous with the mask's 0-bits).

### Best Practices

1. **Use hex for masks** -- `0xFFFFFF00` is immediately readable as "24 bits set." The dotted-decimal equivalent `255.255.255.0` requires mental conversion.

2. **Validate inputs** -- Always check that octets are 0-255, prefixes are 0-32, and masks are contiguous. Networking bugs from bad inputs are notoriously hard to track down.

3. **Prefer CIDR over dotted-decimal masks** -- CIDR notation (`/24`) is unambiguous and compact. Dotted-decimal masks (`255.255.255.0`) can be mistyped as `255.255.255.1`, which is not a valid mask.

4. **Document byte order** -- When storing IPs in files, databases, or network packets, always document whether the value is in host or network byte order.

---

## Related Topics

- [[AND Operator]] -- the core operation for extracting network addresses
- [[OR Operator]] -- used to compute broadcast addresses
- [[NOT Operator]] -- used to compute complement masks and wildcard masks
- [[Left Shift]] -- used to build masks from CIDR prefix lengths
- [[Binary Number System]] -- how binary representation underpins all IP addressing
- [[Hexadecimal and Octal]] -- hex notation for readable IP and mask values
- [[Endianness]] -- network byte order vs host byte order
- [[Bits Bytes and Words]] -- understanding octets, bytes, and 32-bit words
- [[Creating Bit Masks]] -- general techniques for constructing bitmasks
- [[Color and Graphics]] -- another application of packing multiple values into one integer
- [[Permissions and Access Flags]] -- another application of bitwise AND/OR for flag checking
