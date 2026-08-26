---
tags:
  - bit-manipulation
  - bit-flags
  - enum
  - pattern
---

## What Are Bit Flags?

A **bit flag** is a design pattern where individual bits within a single integer each represent an independent boolean value. Instead of storing multiple `bool` variables, you pack them into one number -- each bit acts as a tiny on/off switch.

This technique is foundational to systems programming and appears everywhere: file permissions, window styles, compiler options, hardware registers, network protocol headers, and feature toggles.

The key insight is that **powers of 2** (`1, 2, 4, 8, 16, 32, ...`) are the only integers that have exactly one bit set. This means they never overlap when combined with bitwise OR, making them perfect candidates for flag values.

```
Power of 2    Binary         Bit position
---------    ----------      ------------
  1          0000 0001       bit 0
  2          0000 0010       bit 1
  4          0000 0100       bit 2
  8          0000 1000       bit 3
 16          0001 0000       bit 4
 32          0010 0000       bit 5
 64          0100 0000       bit 6
128          1000 0000       bit 7
```

```ad-warning
title: Sequential Enum Values Cannot Be Combined
If you define enum values as sequential integers (0, 1, 2, 3, 4...), you cannot combine them with bitwise OR and later distinguish them. For example, `1 | 2 = 3`, but `3` is also a valid enum value -- you have no way to tell whether the result means "flag 3 alone" or "flags 1 and 2 combined". Powers of 2 avoid this ambiguity entirely because each value occupies a unique bit.
```

---

## How Multiple Flags Pack Into One Integer

When you OR multiple power-of-2 flags together, each one lights up its own bit without disturbing the others. A single byte can hold 8 independent boolean flags:

```
Bit:       7     6     5     4     3     2     1     0
         +-----+-----+-----+-----+-----+-----+-----+-----+
Value:   |  0  |  0  |  1  |  0  |  0  |  1  |  1  |  0  |  = 0x26 = 38
         +-----+-----+-----+-----+-----+-----+-----+-----+
           |           |                 |     |
           |           |                 |     +-- Write    (1 << 1 = 2)
           |           |                 +-------- Read     (1 << 2 = 4)
           |           +-------------------------- Admin    (1 << 5 = 32)
           +-------------------------------------- (unused, not set)
```

In this example, the integer `38` simultaneously encodes that the `Write`, `Read`, and `Admin` flags are all active. You extracted three booleans from a single number.

```
Building the value step by step:

  Write  = 1 << 1 = 0000 0010  (2)
  Read   = 1 << 2 = 0000 0100  (4)
  Admin  = 1 << 5 = 0010 0000  (32)
                    ----------
  OR all together:  0010 0110  (38)
```

With a 32-bit integer, you can store up to **32 independent flags** in a single variable. A 64-bit integer gives you 64 flags.

---

## Flag Operations Summary

These four operations form the complete toolkit for working with bit flags. Every language uses the same underlying bitwise operators:

| Operation | Expression | What It Does |
|-----------|-----------|-------------|
| **Set** a flag | `flags \|= FLAG` | Turns the flag on (forces bit to 1) |
| **Clear** a flag | `flags &= ~FLAG` | Turns the flag off (forces bit to 0) |
| **Toggle** a flag | `flags ^= FLAG` | Flips the flag (1 becomes 0, 0 becomes 1) |
| **Check** a single flag | `(flags & FLAG) != 0` | Tests whether the flag is set |
| **Check all** in a mask | `(flags & MASK) == MASK` | Tests whether *every* flag in the mask is set |
| **Check any** in a mask | `(flags & MASK) != 0` | Tests whether *at least one* flag in the mask is set |

```ad-tip
title: Check All vs. Check Any
These two checks are easy to confuse. `(flags & MASK) != 0` returns true if *any* bit in the mask is set. `(flags & MASK) == MASK` returns true only if *all* bits in the mask are set. Choose carefully based on your intent.
```

For detailed breakdowns of each operator, see [[AND Operator]], [[OR Operator]], [[XOR Operator]], and [[NOT Operator]].

---

## The "None" and "All" Values Pattern

Most flag enums include two sentinel values:

### None = 0

The value `0` means **no flags are set**. Every bit is `0`. This is the natural "empty" or "default" state.

- `flags & None` is always `0` (AND with zero clears everything)
- `flags | None` is always `flags` (OR with zero changes nothing)
- Useful as an initializer: `var perms = Permission.None;`

### All = every flag combined

The "all" value means **every defined flag is set**. There are two common approaches:

**Approach 1 -- Explicit OR of all flags:**

```csharp
All = Read | Write | Execute | Admin  // only defined flags
```

**Approach 2 -- Bitwise complement of zero:**

```csharp
All = ~0  // every bit set to 1 (0xFFFFFFFF for 32-bit)
```

```ad-warning
title: Be Careful With ~0
Using `~0` sets *every* bit in the integer, including bits that do not correspond to any defined flag. This can cause unexpected behavior in range checks, serialization, or `ToString()` output. The explicit OR approach is safer when you have a fixed set of known flags.
```

---

## C# [Flags] Attribute and Flag Enums

C# provides first-class support for bit flags through the `[Flags]` attribute. This is the most common way to define and work with flag enums in .NET.

### Defining a [Flags] Enum

```csharp
[Flags]
public enum Permission
{
    None    = 0,         // 0000 0000 -- no permissions
    Read    = 1 << 0,    // 0000 0001 -- (1)
    Write   = 1 << 1,    // 0000 0010 -- (2)
    Execute = 1 << 2,    // 0000 0100 -- (4)
    Delete  = 1 << 3,    // 0000 1000 -- (8)
    Admin   = 1 << 4,    // 0001 0000 -- (16)

    // Convenience combinations
    ReadWrite    = Read | Write,             // 0000 0011 -- (3)
    StandardUser = Read | Write | Execute,   // 0000 0111 -- (7)
    All          = Read | Write | Execute | Delete | Admin  // 0001 1111 -- (31)
}
```

```ad-tip
title: Always Use the [Flags] Attribute
The `[Flags]` attribute does not change how bitwise operations work -- the compiler treats the enum the same either way. What it *does* change is the `ToString()` behavior. Without `[Flags]`, calling `ToString()` on a combined value like `3` prints `"3"`. With `[Flags]`, it prints `"Read, Write"`. This is invaluable for debugging and logging.
```

### Full C# Usage Example

```csharp
using System;

[Flags]
public enum Permission
{
    None    = 0,
    Read    = 1 << 0,
    Write   = 1 << 1,
    Execute = 1 << 2,
    Delete  = 1 << 3,
    Admin   = 1 << 4,

    ReadWrite    = Read | Write,
    StandardUser = Read | Write | Execute,
    All          = Read | Write | Execute | Delete | Admin
}

class Program
{
    static void Main()
    {
        // --- Setting flags ---
        var perms = Permission.None;
        perms |= Permission.Read;               // set Read
        perms |= Permission.Write;              // set Write
        Console.WriteLine(perms);               // "Read, Write"

        // Combine multiple at once
        perms = Permission.Read | Permission.Execute | Permission.Admin;
        Console.WriteLine(perms);               // "Read, Execute, Admin"

        // --- Checking flags ---
        // Method 1: HasFlag (readable, slightly slower due to boxing before .NET 7)
        bool canRead = perms.HasFlag(Permission.Read);      // true
        bool canWrite = perms.HasFlag(Permission.Write);    // false

        // Method 2: Bitwise AND (fastest, no boxing)
        bool canExec = (perms & Permission.Execute) != 0;   // true

        // Check if ALL flags in a group are set
        bool isStandard = (perms & Permission.StandardUser) == Permission.StandardUser;
        // false -- Write is missing

        // --- Removing a flag ---
        perms &= ~Permission.Admin;             // clear Admin
        Console.WriteLine(perms);               // "Read, Execute"

        // --- Toggling a flag ---
        perms ^= Permission.Write;              // toggle Write on (was off)
        Console.WriteLine(perms);               // "Read, Write, Execute"
        perms ^= Permission.Write;              // toggle Write off (was on)
        Console.WriteLine(perms);               // "Read, Execute"
    }
}
```

```ad-tip
title: HasFlag vs. Bitwise AND Performance
In .NET Framework and older .NET versions, `HasFlag()` causes boxing because Enum is a value type and HasFlag accepts `Enum` (a reference type). Starting with .NET 7, the JIT intrinsifies `HasFlag()` to a simple bitwise AND, eliminating the boxing. For hot paths on older runtimes, prefer the manual `(flags & FLAG) != 0` pattern.
```

```ad-warning
title: Checking Against None
Never use `perms.HasFlag(Permission.None)` to test for "no flags set". Since `None = 0`, the expression `(perms & 0) == 0` is *always true* regardless of what flags are set. Instead, test `perms == Permission.None` directly.
```

---

## C++ Enum with Bit Values

C++ does not have a built-in `[Flags]` equivalent, so you must be deliberate about how you define and use flag enums.

### Traditional (Unscoped) Enum

Traditional enums implicitly convert to integers, so bitwise operations work out of the box:

```cpp
#include <iostream>
#include <cstdint>

enum Permission : uint8_t {
    PERM_NONE    = 0,
    PERM_READ    = 1 << 0,   // 1
    PERM_WRITE   = 1 << 1,   // 2
    PERM_EXECUTE = 1 << 2,   // 4
    PERM_DELETE  = 1 << 3,   // 8
    PERM_ADMIN   = 1 << 4,   // 16
};

int main() {
    // Combine flags -- result is implicitly an integer
    unsigned flags = PERM_READ | PERM_WRITE | PERM_ADMIN;

    // Check a flag
    if (flags & PERM_READ) {
        std::cout << "Can read\n";
    }

    // Remove a flag
    flags &= ~PERM_ADMIN;

    // Toggle a flag
    flags ^= PERM_EXECUTE;

    return 0;
}
```

### C++11 Scoped Enum (enum class)

Scoped enums (`enum class`) provide type safety but **do not support bitwise operators by default**. You must overload them:

```cpp
#include <iostream>
#include <cstdint>
#include <type_traits>

enum class Permission : uint8_t {
    None    = 0,
    Read    = 1 << 0,
    Write   = 1 << 1,
    Execute = 1 << 2,
    Delete  = 1 << 3,
    Admin   = 1 << 4,
};

// --- Operator overloads for bitwise operations ---

constexpr Permission operator|(Permission lhs, Permission rhs) {
    return static_cast<Permission>(
        static_cast<std::underlying_type_t<Permission>>(lhs) |
        static_cast<std::underlying_type_t<Permission>>(rhs)
    );
}

constexpr Permission operator&(Permission lhs, Permission rhs) {
    return static_cast<Permission>(
        static_cast<std::underlying_type_t<Permission>>(lhs) &
        static_cast<std::underlying_type_t<Permission>>(rhs)
    );
}

constexpr Permission operator^(Permission lhs, Permission rhs) {
    return static_cast<Permission>(
        static_cast<std::underlying_type_t<Permission>>(lhs) ^
        static_cast<std::underlying_type_t<Permission>>(rhs)
    );
}

constexpr Permission operator~(Permission val) {
    return static_cast<Permission>(
        ~static_cast<std::underlying_type_t<Permission>>(val)
    );
}

// Compound assignment operators
inline Permission& operator|=(Permission& lhs, Permission rhs) {
    lhs = lhs | rhs;
    return lhs;
}

inline Permission& operator&=(Permission& lhs, Permission rhs) {
    lhs = lhs & rhs;
    return lhs;
}

inline Permission& operator^=(Permission& lhs, Permission rhs) {
    lhs = lhs ^ rhs;
    return lhs;
}

// Helper to check if a flag is set
constexpr bool hasFlag(Permission flags, Permission test) {
    return (flags & test) == test;
}

int main() {
    Permission perms = Permission::Read | Permission::Write;

    if (hasFlag(perms, Permission::Read)) {
        std::cout << "Can read\n";
    }

    perms &= ~Permission::Write;     // clear Write
    perms ^= Permission::Execute;    // toggle Execute on

    return 0;
}
```

```ad-warning
title: C++ enum class Requires Operator Overloads
If you try to write `Permission::Read | Permission::Write` with a scoped enum and no overloads, you will get a compile error. This is the trade-off for type safety. Many codebases define a macro or template to generate these overloads for any flag enum automatically.
```

```ad-tip
title: Reusable Macro Pattern
Many C++ projects define a macro like `DEFINE_ENUM_FLAG_OPERATORS(EnumType)` that generates all the bitwise overloads at once. The Windows SDK provides `DEFINE_ENUM_FLAG_OPERATORS` in `<winnt.h>` for exactly this purpose.
```

---

## JavaScript Object/Const Pattern

JavaScript has no enum keyword, so bit flags are typically defined using plain objects or `const` declarations:

### Object with Frozen Values

```javascript
const Permission = Object.freeze({
    NONE:    0,
    READ:    1 << 0,   // 1
    WRITE:   1 << 1,   // 2
    EXECUTE: 1 << 2,   // 4
    DELETE:  1 << 3,   // 8
    ADMIN:   1 << 4,   // 16
});

// Combine flags
let perms = Permission.READ | Permission.WRITE;

// Check a flag
if (perms & Permission.READ) {
    console.log("Can read");
}

// Check all flags in a group
const editorPerms = Permission.READ | Permission.WRITE;
if ((perms & editorPerms) === editorPerms) {
    console.log("Has full editor permissions");
}

// Remove a flag
perms &= ~Permission.WRITE;

// Toggle a flag
perms ^= Permission.EXECUTE;

// Set a flag
perms |= Permission.ADMIN;
```

```ad-warning
title: JavaScript Bitwise Operators Use 32-bit Signed Integers
All bitwise operators in JavaScript convert their operands to **32-bit signed integers** before operating, even though JavaScript numbers are 64-bit floats. This means you are limited to 31 usable flag bits (bit 31 is the sign bit). If you need more than 31 flags, use `BigInt` with the `n` suffix on literals.
```

### TypeScript Enum Approach

TypeScript supports const enums which compile away to inline constants:

```javascript
// TypeScript
const enum Permission {
    None    = 0,
    Read    = 1 << 0,
    Write   = 1 << 1,
    Execute = 1 << 2,
    Delete  = 1 << 3,
    Admin   = 1 << 4,
}

let perms: number = Permission.Read | Permission.Write;
```

---

## Common Operations in Detail

### Setting Multiple Flags at Once

```
Before:   0000 0001  (Read only)
          0000 0110  (Write | Execute mask)
          ---------
OR:       0000 0111  (Read | Write | Execute)
```

You can combine any number of flags in a single expression:

```csharp
var perms = Permission.Read | Permission.Write | Permission.Execute;
```

### Clearing Multiple Flags at Once

```
Before:   0001 0111  (Read | Write | Execute | Admin)
Mask:     0000 0110  (Write | Execute)
~Mask:    1111 1001
          ---------
AND:      0001 0001  (Read | Admin)
```

```csharp
perms &= ~(Permission.Write | Permission.Execute);  // clear both at once
```

### Testing If Any of Several Flags Are Set

```csharp
// True if the user has Read OR Write (or both)
bool canAccess = (perms & (Permission.Read | Permission.Write)) != 0;
```

### Testing If All of Several Flags Are Set

```csharp
// True only if the user has BOTH Read AND Write
var required = Permission.Read | Permission.Write;
bool hasAll = (perms & required) == required;
```

---

## Bit Index vs. Flag Value

A common source of confusion: the **bit index** (position) and the **flag value** are different numbers.

```
Bit index:   0     1     2     3     4     5     6     7
Flag value:  1     2     4     8    16    32    64   128
Shift expr:  1<<0  1<<1  1<<2  1<<3 1<<4  1<<5  1<<6  1<<7
```

```ad-warning
title: Off-by-One with Shifts
A common mistake is writing `1 << 1` for the first flag, skipping bit 0 entirely. This wastes a bit position and can cause confusion when debugging. Convention is to start with `1 << 0` for the first flag. Alternatively, just write the literal value `1` for the first flag.
```

---

## Real-World Examples

### File Permissions (Unix-style)

The classic example of bit flags in practice. See [[Permissions and Access Flags]] for the full treatment.

```
Owner    Group    Other
rwx      rwx      rwx
|||      |||      |||
421      421      421

chmod 755 = 111 101 101
  Owner: read + write + execute  = 7
  Group: read + execute          = 5
  Other: read + execute          = 5
```

### UI Component State

```csharp
[Flags]
enum WidgetState
{
    None     = 0,
    Visible  = 1 << 0,   // 1
    Enabled  = 1 << 1,   // 2
    Focused  = 1 << 2,   // 4
    Selected = 1 << 3,   // 8
    Hovered  = 1 << 4,   // 16
    Dirty    = 1 << 5,   // 32 -- has unsaved changes

    Default  = Visible | Enabled,  // normal interactive state
}
```

### Feature Toggles

```csharp
[Flags]
enum Feature
{
    None          = 0,
    DarkMode      = 1 << 0,
    Notifications = 1 << 1,
    Analytics     = 1 << 2,
    BetaFeatures  = 1 << 3,
    OfflineMode   = 1 << 4,
}

// Enable features per user
var userFeatures = Feature.DarkMode | Feature.Notifications;

// Check before showing UI
if (userFeatures.HasFlag(Feature.BetaFeatures))
{
    ShowBetaUI();
}
```

---

## When to Use Bit Flags vs. Other Patterns

Bit flags are powerful but not always the best choice. Consider the trade-offs:

| Scenario | Bit Flags? | Alternative |
|----------|-----------|-------------|
| Multiple independent on/off options | Yes | -- |
| Options are mutually exclusive (only one at a time) | No | Regular enum |
| More than 32/64 options | No | `HashSet<T>`, `BitArray` |
| Options need associated data (not just on/off) | No | Dictionary, class hierarchy |
| Performance-critical hot path | Yes | -- |
| Needs to be human-readable in config files | Maybe | String arrays |
| Needs to be stored in a single DB column | Yes | -- |
| Cross-language serialization | Careful | Explicit mapping |

```ad-tip
title: Database Storage
Bit flags are convenient for databases because a single integer column can represent many boolean properties. A query like `WHERE (permissions & 4) != 0` checks the Read flag directly in SQL. But be aware that this approach defeats indexing on individual flags -- if you frequently query by a single flag, a separate boolean column may perform better.
```

---

## See Also

- [[Creating Bit Masks]] -- constructing masks for isolating, clearing, and modifying groups of bits
- [[AND Operator]] -- the operator used to test and clear flags
- [[OR Operator]] -- the operator used to set and combine flags
- [[XOR Operator]] -- the operator used to toggle flags
- [[NOT Operator]] -- the operator used to invert masks for clearing flags
- [[Permissions and Access Flags]] -- real-world permission systems built on bit flags
- [[Check Set and Clear Bits]] -- fundamental bit inspection and modification techniques
- [[Toggle Bits]] -- detailed coverage of XOR-based toggling patterns
