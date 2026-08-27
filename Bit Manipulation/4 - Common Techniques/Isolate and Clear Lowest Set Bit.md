---
tags:
  - bit-manipulation
  - technique
  - lowest-set-bit
---

## Isolate and Clear Lowest Set Bit

The lowest set bit (also called the rightmost set bit or least significant set bit) is the `1` bit at the smallest position in a binary number. Two closely related operations -- **isolating** and **clearing** this bit -- are fundamental building blocks used in [[Count Set Bits|Kernighan's bit counting algorithm]], [[Check Power of Two|power-of-two checks]], and many other bit manipulation patterns.

---

## Isolate Lowest Set Bit

**Formula:** `n & (-n)`

Equivalently: `n & (~n + 1)`

This produces a value with **only** the lowest set bit of `n` turned on -- all other bits are zero.

### Why It Works

In [[Twos Complement]], `-n` is computed as `~n + 1` (invert all bits, then add 1). Here is what happens step by step:

```
n       = ...1 0 1 0 0 0     (some number with lowest set bit at position 3)
               ^
         lowest set bit

~n      = ...0 1 0 1 1 1     (all bits inverted)

~n + 1  = ...0 1 1 0 0 0     (add 1: carries ripple through the trailing 1s,
               ^               stops at the position of n's lowest set bit)
         this bit becomes 1

-n      = ...0 1 1 0 0 0     (this is -n in two's complement)
```

Now AND `n` with `-n`:

```
  n     = ...1 0 1 0 0 0
  -n    = ...0 1 1 0 0 0
           --|---|-------
  n & -n = 0 0 1 0 0 0 0     only the lowest set bit survives
                 ^
```

**The key insight:** Below the lowest set bit, `n` has all `0`s and `-n` has all `0`s (after the carry ripple). Above the lowest set bit, `n` and `-n` are complements of each other (one has `0` where the other has `1`). Only at the lowest set bit position do both have `1`.

### ASCII Art Walkthrough

**Example: n = 52 (`0b00110100`)**

```
n       =  0  0  1  1  0  1  0  0     (52)
                          ^
                   lowest set bit (bit 2)

~n      =  1  1  0  0  1  0  1  1     (bitwise NOT)

~n + 1  =  1  1  0  0  1  1  0  0     (add 1: ripple carry through bits 0,1)
                          ^
                   this bit stays 1

-n      =  1  1  0  0  1  1  0  0     (-52 in two's complement)

n & -n:
    00110100
  & 11001100
  ----------
    00000100     = 4   (only bit 2 is set -- the lowest set bit of 52)
```

**Example: n = 80 (`0b01010000`)**

```
n       =  0  1  0  1  0  0  0  0     (80)
                       ^
                lowest set bit (bit 4)

-n      =  1  0  1  1  0  0  0  0     (-80)

n & -n:
    01010000
  & 10110000
  ----------
    00010000     = 16  (only bit 4 is set)
```

---

## Clear Lowest Set Bit

**Formula:** `n & (n - 1)`

This turns off the lowest set bit and leaves all other bits unchanged.

### Why It Works

Subtracting 1 from `n` flips the lowest set bit to `0` and turns all the bits below it to `1`:

```
n       = ...1 0 1 0 0 0     (lowest set bit at position 3)
               ^

n - 1   = ...1 0 0 1 1 1     (borrow ripples down from the lowest set bit)
               ^  ^--^--^
        this bit  these bits
        cleared   all become 1
```

ANDing `n` with `n - 1`:

```
  n     = ...1 0 1 0 0 0
  n - 1 = ...1 0 0 1 1 1
           --|---|-|-----|
  n&(n-1) = 1 0 0 0 0 0     lowest set bit cleared, all others preserved
```

Bits above the lowest set bit are identical in `n` and `n - 1`, so they survive the AND. The lowest set bit itself and all bits below it differ, so they become zero.

### ASCII Art Walkthrough

**Example: n = 52 (`0b00110100`)**

```
n       =  0  0  1  1  0  1  0  0     (52)
                          ^
                   lowest set bit (bit 2)

n - 1   =  0  0  1  1  0  0  1  1     (51)
                          ^  ^--^
                    cleared  flipped to 1

n & (n-1):
    00110100
  & 00110011
  ----------
    00110000     = 48   (bit 2 cleared, rest preserved)
```

**Example: n = 80 (`0b01010000`)**

```
n       =  0  1  0  1  0  0  0  0     (80)
                       ^
                lowest set bit (bit 4)

n - 1   =  0  1  0  0  1  1  1  1     (79)

n & (n-1):
    01010000
  & 01001111
  ----------
    01000000     = 64   (bit 4 cleared)
```

```ad-tip
title: Connection to Kernighan's Algorithm
[[Count Set Bits|Kernighan's bit counting algorithm]] repeatedly applies `n = n & (n - 1)` to clear the lowest set bit one at a time, counting how many iterations it takes to reach zero. Each iteration removes exactly one set bit.
```

---

## Set Lowest Clear Bit

**Formula:** `n | (n + 1)`

This turns on the lowest `0` bit (the rightmost clear bit) and leaves all other bits unchanged.

### Why It Works

Adding 1 to `n` causes a carry that ripples through all trailing `1` bits and flips the first `0` it encounters:

```
n       = ...0  1  1  1     (lowest clear bit at position 3)
              ^
        lowest clear bit

n + 1   = ...1  0  0  0     (carry ripples through trailing 1s)
              ^
        this bit set to 1

n | (n+1):
    ...0 1 1 1
  | ...1 0 0 0
  = ...1 1 1 1     lowest clear bit is now set, others preserved
```

### ASCII Art Walkthrough

**Example: n = 39 (`0b00100111`)**

```
n       =  0  0  1  0  0  1  1  1     (39)
                    ^
             lowest clear bit (bit 3)

n + 1   =  0  0  1  0  1  0  0  0     (40)

n | (n+1):
    00100111
  | 00101000
  ----------
    00101111     = 47   (bit 3 set, rest preserved)
```

---

## Code Examples

### C\#

```csharp
public static class LowestBitOps
{
    /// <summary>Isolate the lowest set bit: returns a value with only that bit on.</summary>
    public static int IsolateLowestSetBit(int n)
    {
        return n & (-n);
    }

    /// <summary>Clear the lowest set bit.</summary>
    public static int ClearLowestSetBit(int n)
    {
        return n & (n - 1);
    }

    /// <summary>Set the lowest clear bit.</summary>
    public static int SetLowestClearBit(int n)
    {
        return n | (n + 1);
    }

    /// <summary>Get the position (index) of the lowest set bit.</summary>
    public static int LowestSetBitPosition(int n)
    {
        if (n == 0) return -1;  // no set bit

        int isolated = n & (-n);
        int position = 0;
        while ((isolated >>= 1) != 0)
            position++;

        return position;
    }
}

// Usage
int n = 52;  // 0b00110100
Console.WriteLine($"n = {Convert.ToString(n, 2).PadLeft(8, '0')}");

int isolated = LowestBitOps.IsolateLowestSetBit(n);
Console.WriteLine($"Isolate lowest set bit: {Convert.ToString(isolated, 2).PadLeft(8, '0')} ({isolated})");
// 00000100 (4)

int cleared = LowestBitOps.ClearLowestSetBit(n);
Console.WriteLine($"Clear lowest set bit:   {Convert.ToString(cleared, 2).PadLeft(8, '0')} ({cleared})");
// 00110000 (48)

int withBitSet = LowestBitOps.SetLowestClearBit(n);
Console.WriteLine($"Set lowest clear bit:   {Convert.ToString(withBitSet, 2).PadLeft(8, '0')} ({withBitSet})");
// 00110101 (53)

Console.WriteLine($"Lowest set bit position: {LowestBitOps.LowestSetBitPosition(n)}");
// 2
```

### C++

```cpp
#include <iostream>
#include <bitset>

// Isolate the lowest set bit
int isolateLowestSetBit(int n) {
    return n & (-n);
}

// Clear the lowest set bit
int clearLowestSetBit(int n) {
    return n & (n - 1);
}

// Set the lowest clear bit
int setLowestClearBit(int n) {
    return n | (n + 1);
}

// Get the position of the lowest set bit
// GCC/Clang: __builtin_ctz() counts trailing zeros
int lowestSetBitPosition(int n) {
    if (n == 0) return -1;
    return __builtin_ctz(static_cast<unsigned>(n));
}

int main() {
    int n = 52;  // 0b00110100
    std::cout << "n = " << std::bitset<8>(n) << "\n";

    int isolated = isolateLowestSetBit(n);
    std::cout << "Isolate: " << std::bitset<8>(isolated)
              << " (" << isolated << ")\n";

    int cleared = clearLowestSetBit(n);
    std::cout << "Clear:   " << std::bitset<8>(cleared)
              << " (" << cleared << ")\n";

    int withSet = setLowestClearBit(n);
    std::cout << "Set:     " << std::bitset<8>(withSet)
              << " (" << withSet << ")\n";

    std::cout << "Position: " << lowestSetBitPosition(n) << "\n";

    return 0;
}
```

### JavaScript

```javascript
// Isolate the lowest set bit
function isolateLowestSetBit(n) {
    return n & (-n);
}

// Clear the lowest set bit
function clearLowestSetBit(n) {
    return n & (n - 1);
}

// Set the lowest clear bit
function setLowestClearBit(n) {
    return n | (n + 1);
}

// Get the position of the lowest set bit
function lowestSetBitPosition(n) {
    if (n === 0) return -1;
    let isolated = n & (-n);
    return Math.log2(isolated);
    // Or use: 31 - Math.clz32(isolated)
}

// Usage
const n = 52;  // 0b00110100
console.log(`n = ${(n >>> 0).toString(2).padStart(8, '0')}`);

const isolated = isolateLowestSetBit(n);
console.log(`Isolate: ${(isolated >>> 0).toString(2).padStart(8, '0')} (${isolated})`);
// 00000100 (4)

const cleared = clearLowestSetBit(n);
console.log(`Clear:   ${(cleared >>> 0).toString(2).padStart(8, '0')} (${cleared})`);
// 00110000 (48)

const withSet = setLowestClearBit(n);
console.log(`Set:     ${(withSet >>> 0).toString(2).padStart(8, '0')} (${withSet})`);
// 00110101 (53)

console.log(`Position: ${lowestSetBitPosition(n)}`);
// 2
```

---

## Use Cases

### Iterating Over Set Bits

Combining isolation and clearing lets you enumerate all set bit positions:

```csharp
void PrintSetBits(int n)
{
    while (n != 0)
    {
        int lowestBit = n & (-n);            // Isolate lowest set bit
        int position = BitOperations.TrailingZeroCount((uint)lowestBit);
        Console.WriteLine($"Bit {position} is set");
        n &= (n - 1);                        // Clear it and move on
    }
}

PrintSetBits(0b10110100);
// Bit 2 is set
// Bit 4 is set
// Bit 5 is set
// Bit 7 is set
```

### Fenwick Trees (Binary Indexed Trees)

Fenwick trees use `n & (-n)` to navigate between tree nodes. The lowest set bit determines the range of each node, making these operations essential for efficient prefix sum queries.

```ad-warning
title: Watch for n = 0
Both `n & (-n)` and `n & (n - 1)` return 0 when `n = 0`. Make sure to handle the zero case explicitly if your algorithm requires it (e.g., `lowestSetBitPosition` returning -1 for zero).
```

---

## Summary

| Operation              | Formula          | Result                                      |
| ---------------------- | ---------------- | ------------------------------------------- |
| Isolate lowest set bit | `n & (-n)`       | Value with only the lowest set bit           |
| Clear lowest set bit   | `n & (n - 1)`   | Value with lowest set bit turned off         |
| Set lowest clear bit   | `n \| (n + 1)`  | Value with lowest clear bit turned on        |

---

## Related Concepts

- **[[Twos Complement]]** -- explains why `-n = ~n + 1` and makes `n & (-n)` work.
- **[[NOT Operator]]** -- `~n` is the first step in computing `-n`.
- **[[AND Operator]]** -- the operator that performs the isolation/clearing.
- **[[Count Set Bits]]** -- Kernighan's algorithm is built on `n & (n - 1)`.
- **[[Check Power of Two]]** -- `n & (n - 1) == 0` tests for exactly one set bit.
