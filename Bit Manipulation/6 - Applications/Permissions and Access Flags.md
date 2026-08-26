---
title: Permissions and Access Flags
tags:
  - bit-manipulation
  - application
  - permissions
  - flags
aliases:
  - Permission Bits
  - Access Control Flags
  - Permission Bitmasks
date: 2026-08-19
---

## Permissions and Access Flags

One of the most widespread real-world applications of [[Binary Number System|binary]] and bitwise operations is **permission systems**. From Unix file permissions to application-level access control, bit flags provide a compact, fast, and elegant way to represent, combine, and check sets of permissions. Every permission maps to a single bit, and a user's complete access profile fits inside a single integer.

This note covers the Unix permission model, how to design your own permission system using [[Creating Bit Masks|bitmasks]], role-based access control with bit flags, and complete code examples in C#, C++, and JavaScript.

---

## Unix File Permissions Model

Unix file permissions are the textbook example of bit manipulation in production systems. Every file and directory has a **9-bit permission field** that controls who can read, write, and execute it.

### The Three Permission Types

Each permission type is assigned a distinct power of 2 so that they never overlap when combined:

| Permission | Symbol | Decimal | Binary | Bit Position |
|:----------:|:------:|:-------:|:------:|:------------:|
| Read       | `r`    | 4       | `100`  | 2            |
| Write      | `w`    | 2       | `010`  | 1            |
| Execute    | `x`    | 1       | `001`  | 0            |

Because each permission occupies a unique bit, you can combine them freely with [[OR Operator|OR]] and test them with [[AND Operator|AND]] without any ambiguity:

- **Read + Write** = `4 | 2` = `6` = `110`
- **Read + Execute** = `4 | 1` = `5` = `101`
- **All three** = `4 | 2 | 1` = `7` = `111`

```ad-note
title: Why Powers of 2?
Each power of 2 has exactly one bit set in its binary representation. This guarantees that combining permissions with OR never causes "collisions" — each permission is independently addressable. This is the foundational principle behind all bit flag systems. See [[Bits Bytes and Words]] for more on binary representation.
```

### Owner, Group, and Other

Unix splits permissions into three **categories**, each getting its own 3-bit field:

| Category | Description                          | Bit Range |
|:--------:|--------------------------------------|:---------:|
| Owner    | The file's creator/owner             | Bits 8–6  |
| Group    | Users in the file's assigned group   | Bits 5–3  |
| Other    | Everyone else on the system          | Bits 2–0  |

These three 3-bit fields are packed together into a single 9-bit value. Here is the complete layout:

```
  Owner       Group       Other
┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ r │ w │ x │ r │ w │ x │ r │ w │ x │
├───┼───┼───┼───┼───┼───┼───┼───┼───┤
│ 8 │ 7 │ 6 │ 5 │ 4 │ 3 │ 2 │ 1 │ 0 │  ← bit positions
└───┴───┴───┴───┴───┴───┴───┴───┴───┘
  4   2   1   4   2   1   4   2   1    ← decimal weight within group
```

Each 3-bit group is independently interpreted as an octal digit (0–7), which is why Unix permissions use **octal notation** (see [[Hexadecimal and Octal]]).

### Octal Notation and Common Permission Patterns

Because each octal digit maps perfectly to a 3-bit group, permissions are written as three octal digits:

```
0o755 = 111 101 101
        rwx r-x r-x
        ─┬─ ─┬─ ─┬─
         │   │   └── Other:  read + execute
         │   └────── Group:  read + execute
         └────────── Owner:  read + write + execute
```

Common permission values:

| Octal   | Binary      | Symbolic     | Meaning                                    |
|:-------:|:-----------:|:------------:|--------------------------------------------|
| `0o777` | `111111111` | `rwxrwxrwx`  | Full access for everyone                   |
| `0o755` | `111101101` | `rwxr-xr-x`  | Owner full, others read+execute            |
| `0o644` | `110100100` | `rw-r--r--`  | Owner read+write, others read-only         |
| `0o700` | `111000000` | `rwx------`  | Owner full, no access for anyone else      |
| `0o600` | `110000000` | `rw-------`  | Owner read+write only                      |
| `0o444` | `100100100` | `r--r--r--`  | Read-only for everyone                     |

### Extracting Permissions with Masks

To check a specific category's permissions, you use [[AND Operator|AND]] with a [[Creating Bit Masks|mask]] to isolate the relevant bits, then shift them down:

```cpp
// Extract group permissions from a 9-bit permission value
int permissions = 0755;  // octal: rwxr-xr-x
int groupMask   = 0070;  // octal: isolate bits 5-3
int groupPerms  = (permissions & groupMask) >> 3;
// groupPerms == 5 (r-x)
```

To check if the owner has write permission:

```cpp
bool ownerCanWrite = (permissions & 0200) != 0;
// 0200 octal = bit 7 = owner write bit
```

```ad-tip
title: Memorize the Octal Digits
Each octal digit 0–7 maps directly to a 3-bit pattern. If you memorize the eight combinations, you can read any Unix permission string instantly:
- `0` = `---`, `1` = `--x`, `2` = `-w-`, `3` = `-wx`
- `4` = `r--`, `5` = `r-x`, `6` = `rw-`, `7` = `rwx`
```

```ad-warning
title: Special Permission Bits
Unix actually has a 12-bit permission field, not 9. The top 3 bits are **setuid** (bit 11), **setgid** (bit 10), and the **sticky bit** (bit 9). These are encoded in the leading octal digit: `0o4755` sets the setuid bit. Forgetting about these extra bits is a common source of security bugs when implementing permission checks manually.
```

---

## Designing a Permission System with Bit Flags

Building your own permission system with bit flags follows the same principles as Unix permissions but can be customized to any domain. The core idea: **each permission is a named constant equal to a unique power of 2**, and the user's permissions are a single integer that combines them.

### Defining Permission Constants

Every permission must be a distinct power of 2 so that each occupies a unique bit position (see [[Flag Enums and Bit Flags]]):

```
Permission       Decimal   Binary
─────────────────────────────────────
NONE             0         00000000
READ             1         00000001
WRITE            2         00000010
DELETE           4         00000100
CREATE           8         00001000
ADMIN           16         00010000
EXECUTE         32         00100000
SHARE           64         01000000
AUDIT          128         10000000
```

```ad-warning
title: Never Use Non-Power-of-2 Values
A common mistake is defining permissions as sequential integers (1, 2, 3, 4...) instead of powers of 2 (1, 2, 4, 8...). If you use 3 as a permission value, it occupies **two** bits (`011`) and becomes indistinguishable from the combination of 1 and 2. This breaks all flag logic. Always use `1 << n` or explicit powers of 2. See [[Check Set and Clear Bits]] for more.
```

### The Four Core Operations

Every permission system needs exactly four bitwise operations. These are the building blocks for all access control logic:

#### Granting a Permission (Set Bits with OR)

To add a permission to a user's existing set, use [[OR Operator|OR]]:

```cpp
userPerms |= WRITE;           // Grant write access
userPerms |= READ | EXECUTE;  // Grant multiple at once
```

The [[OR Operator|OR]] operation sets the target bit(s) to 1 without affecting any other bits. If the permission was already granted, OR-ing it again is harmless — the bit stays 1.

#### Revoking a Permission (Clear Bits with AND NOT)

To remove a permission, use [[AND Operator|AND]] with the [[NOT Operator|NOT (complement)]] of the permission:

```cpp
userPerms &= ~WRITE;           // Revoke write access
userPerms &= ~(READ | WRITE);  // Revoke multiple at once
```

The `~WRITE` flips all bits, so the WRITE bit becomes 0 and all others become 1. AND-ing with this mask clears the WRITE bit while preserving everything else. See [[Combining and Extracting with Masks]] for the detailed mechanics.

#### Checking for a Specific Permission (Test with AND)

To check if a user has a specific permission, use [[AND Operator|AND]]:

```cpp
bool canWrite = (userPerms & WRITE) != 0;
```

The AND isolates the WRITE bit. If it was set, the result is non-zero (true). If it was clear, the result is zero (false).

#### Toggling a Permission (Flip with XOR)

To flip a permission on or off without knowing its current state:

```cpp
userPerms ^= WRITE;  // Toggle write: on→off or off→on
```

```ad-tip
title: Toggle is Rarely Used in Production
While XOR toggling is elegant, production permission systems almost never use it. You typically know whether you want to grant or revoke, and toggling introduces subtle bugs when the current state is uncertain. Prefer explicit grant (OR) and revoke (AND NOT) operations.
```

### Checking for ANY vs ALL Permissions

This is a critical distinction that many developers get wrong. There are two different questions you can ask:

**Does the user have ANY of these permissions?** (at least one)

```cpp
bool hasAny = (userPerms & requiredMask) != 0;
```

This is true if *at least one* bit in `requiredMask` is also set in `userPerms`. Useful for "can this user do at least one of read, write, or delete?"

**Does the user have ALL of these permissions?** (every single one)

```cpp
bool hasAll = (userPerms & requiredMask) == requiredMask;
```

This is true only if *every* bit in `requiredMask` is also set in `userPerms`. Useful for "this operation requires both read AND write — does the user have both?"

```ad-warning
title: The "!= 0" vs "== mask" Trap
The most common bug in permission systems is using `!= 0` (ANY check) when you meant `== mask` (ALL check). If an operation requires both READ and WRITE, and you write `(perms & (READ | WRITE)) != 0`, a user with only READ access will pass the check. Always use `== mask` when all permissions in the mask are required.
```

### Visualizing the Operations

Here is a visual walkthrough of the four core operations on an 8-bit permission value:

```
Starting permissions: 00000101  (READ | DELETE)

GRANT WRITE (|= 00000010):
  00000101   user perms
| 00000010   WRITE
──────────
  00000111   result: READ | WRITE | DELETE

REVOKE READ (&= ~00000001):
  00000111   user perms
& 11111110   ~READ
──────────
  00000110   result: WRITE | DELETE

CHECK DELETE (& 00000100):
  00000110   user perms
& 00000100   DELETE
──────────
  00000100   non-zero → user HAS delete permission

CHECK READ (& 00000001):
  00000110   user perms
& 00000001   READ
──────────
  00000000   zero → user does NOT have read permission
```

---

## Role-Based Access Control with Bitmasks

In real applications, you rarely assign permissions individually. Instead, you define **roles** — named combinations of permissions — and assign roles to users. Each role is simply a pre-built bitmask.

### Defining Roles as Permission Combinations

```
Role         Permissions                      Bitmask   Decimal
──────────────────────────────────────────────────────────────────
VIEWER       READ                             00000001      1
EDITOR       READ | WRITE | CREATE            00001011     11
MODERATOR    READ | WRITE | DELETE | SHARE    01000111     71
ADMIN        READ | WRITE | DELETE |          11111111    255
             CREATE | ADMIN | EXECUTE |
             SHARE | AUDIT (all permissions)
```

The [[OR Operator|OR]] operator naturally combines any set of permissions into a role bitmask:

```cpp
const int VIEWER    = READ;
const int EDITOR    = READ | WRITE | CREATE;
const int MODERATOR = READ | WRITE | DELETE | SHARE;
const int ADMIN     = READ | WRITE | DELETE | CREATE
                    | ADMIN_PERM | EXECUTE | SHARE | AUDIT;
```

### Role Hierarchy and Inheritance

Because roles are just bitmasks, you can build hierarchies where each level inherits the previous level's permissions:

```cpp
const int VIEWER    = READ;
const int EDITOR    = VIEWER | WRITE | CREATE;        // inherits VIEWER
const int MODERATOR = EDITOR | DELETE | SHARE;         // inherits EDITOR
const int ADMIN     = MODERATOR | ADMIN_PERM | EXECUTE | AUDIT; // inherits all
```

This approach guarantees that each higher role is a **superset** of the lower roles. You can verify hierarchy with a simple check:

```cpp
// Returns true if roleA includes all permissions of roleB
bool inheritsFrom = (roleA & roleB) == roleB;
```

```ad-tip
title: Use ALL_PERMISSIONS for Admin
Instead of manually OR-ing every permission for the admin role, you can define an ALL constant. If you have 8 permissions (bits 0–7), then `ALL = 0xFF` or `ALL = (1 << NUM_PERMISSIONS) - 1`. This automatically includes any new permissions you add later.
```

### Checking Role Membership

You can check if a user's permissions match a role exactly, or if they have *at least* the permissions of a role:

```cpp
// Exact role check (user has these permissions and ONLY these)
bool isExactlyEditor = (userPerms == EDITOR);

// Minimum role check (user has at least these permissions, maybe more)
bool isAtLeastEditor = (userPerms & EDITOR) == EDITOR;
```

The "at least" check is more common — an Admin should pass an "is at least Editor" check.

### Combining Multiple Roles

If a user holds multiple roles, simply OR them together:

```cpp
int userPerms = EDITOR | MODERATOR;
// userPerms = (READ | WRITE | CREATE) | (READ | WRITE | DELETE | SHARE)
// userPerms = READ | WRITE | CREATE | DELETE | SHARE
```

The OR operation handles overlapping permissions gracefully — bits that both roles share (like READ) are set once and stay set.

```ad-note
title: When Bitmask RBAC Falls Short
Bitmask-based RBAC works well when you have a fixed, moderate set of permissions (up to 32 or 64, depending on your integer size). If you need **context-sensitive** permissions (e.g., "can edit only their own posts"), **attribute-based** access control, or hundreds of fine-grained permissions, bitmasks become unwieldy. At that scale, consider a database-driven permission system or a policy engine. The bit flag approach is best for well-defined, global permission sets.
```

---

## Code Examples

The following examples implement a complete mini permission system in C#, C++, and JavaScript. Each example defines permission constants, creates users with combined permissions, checks access, and demonstrates granting and revoking.

### C# — Using the [Flags] Enum Pattern

C# has first-class support for bit flag enums with the `[Flags]` attribute. This is the idiomatic way to implement permissions in C#. See [[Flag Enums and Bit Flags]] for a deep dive on the pattern.

```csharp
using System;

[Flags]
public enum Permission
{
    None    = 0,         // 00000000
    Read    = 1 << 0,    // 00000001
    Write   = 1 << 1,    // 00000010
    Delete  = 1 << 2,    // 00000100
    Create  = 1 << 3,    // 00001000
    Admin   = 1 << 4,    // 00010000
    Execute = 1 << 5,    // 00100000
    Share   = 1 << 6,    // 01000000
    Audit   = 1 << 7,    // 10000000

    // Role definitions as named combinations
    Viewer    = Read,
    Editor    = Read | Write | Create,
    Moderator = Read | Write | Delete | Share,
    FullAccess = Read | Write | Delete | Create
               | Admin | Execute | Share | Audit
}

public class User
{
    public string Name { get; }
    public Permission Permissions { get; private set; }

    public User(string name, Permission permissions)
    {
        Name = name;
        Permissions = permissions;
    }

    // Check if user has ALL specified permissions
    public bool HasPermission(Permission required)
    {
        return (Permissions & required) == required;
    }

    // Check if user has ANY of the specified permissions
    public bool HasAnyPermission(Permission mask)
    {
        return (Permissions & mask) != Permission.None;
    }

    // Grant additional permissions
    public void Grant(Permission perm)
    {
        Permissions |= perm;
    }

    // Revoke specific permissions
    public void Revoke(Permission perm)
    {
        Permissions &= ~perm;
    }

    public void PrintPermissions()
    {
        Console.WriteLine($"{Name}: {Permissions} ({(int)Permissions})");
        Console.WriteLine($"  Binary: {Convert.ToString((int)Permissions, 2).PadLeft(8, '0')}");
    }
}

public class Program
{
    public static void Main()
    {
        // Create users with role-based permissions
        var viewer = new User("Alice", Permission.Viewer);
        var editor = new User("Bob",   Permission.Editor);
        var admin  = new User("Carol", Permission.FullAccess);

        viewer.PrintPermissions();
        editor.PrintPermissions();
        admin.PrintPermissions();

        // Check specific permissions
        Console.WriteLine($"\nBob can write: {editor.HasPermission(Permission.Write)}");       // True
        Console.WriteLine($"Bob can delete: {editor.HasPermission(Permission.Delete)}");       // False
        Console.WriteLine($"Bob can read AND write: "
            + $"{editor.HasPermission(Permission.Read | Permission.Write)}");                  // True

        // Check ANY permission
        Console.WriteLine($"Alice can read OR write: "
            + $"{viewer.HasAnyPermission(Permission.Read | Permission.Write)}");               // True

        // Grant and revoke
        Console.WriteLine("\n--- Granting Bob delete permission ---");
        editor.Grant(Permission.Delete);
        Console.WriteLine($"Bob can delete: {editor.HasPermission(Permission.Delete)}");       // True
        editor.PrintPermissions();

        Console.WriteLine("\n--- Revoking Bob's write permission ---");
        editor.Revoke(Permission.Write);
        Console.WriteLine($"Bob can write: {editor.HasPermission(Permission.Write)}");         // False
        editor.PrintPermissions();

        // C# [Flags] gives you nice ToString() output automatically
        Console.WriteLine($"\nCarol's permissions: {admin.Permissions}");
        // Output: Read, Write, Delete, Create, Admin, Execute, Share, Audit
    }
}
```

```ad-tip
title: [Flags] Enum Best Practices in C#
1. Always include a `None = 0` member — it represents no permissions.
2. Always use `1 << n` syntax instead of raw numbers — it is self-documenting and less error-prone.
3. The `[Flags]` attribute enables `ToString()` to print combined names (e.g., `"Read, Write"` instead of `"3"`).
4. Use `HasFlag()` as an alternative: `perms.HasFlag(Permission.Write)` — but note that `HasFlag` was slower than manual AND in older .NET versions. In .NET Core 3.0+ it is JIT-optimized to a single AND instruction.
5. Named combinations (like `Editor = Read | Write | Create`) are recognized by `ToString()` and make the API self-documenting.
```

### C++ — Bit Flag Constants and Bitwise Operations

C++ does not have a built-in `[Flags]` attribute, but you can use `enum` with operator overloading or plain constants. Here is a clean approach using `enum class` with overloaded operators:

```cpp
#include <iostream>
#include <string>
#include <cstdint>

// Define permissions as an enum class with explicit underlying type
enum class Permission : uint8_t
{
    None    = 0,         // 00000000
    Read    = 1 << 0,    // 00000001
    Write   = 1 << 1,    // 00000010
    Delete  = 1 << 2,    // 00000100
    Create  = 1 << 3,    // 00001000
    Admin   = 1 << 4,    // 00010000
    Execute = 1 << 5,    // 00100000
    Share   = 1 << 6,    // 01000000
    Audit   = 1 << 7,    // 10000000

    // Role definitions
    Viewer    = Read,
    Editor    = static_cast<uint8_t>(Read) | static_cast<uint8_t>(Write)
              | static_cast<uint8_t>(Create),
    Moderator = static_cast<uint8_t>(Read) | static_cast<uint8_t>(Write)
              | static_cast<uint8_t>(Delete) | static_cast<uint8_t>(Share),
    FullAccess = 0xFF
};

// Overload bitwise operators for Permission
inline Permission operator|(Permission a, Permission b) {
    return static_cast<Permission>(
        static_cast<uint8_t>(a) | static_cast<uint8_t>(b));
}
inline Permission operator&(Permission a, Permission b) {
    return static_cast<Permission>(
        static_cast<uint8_t>(a) & static_cast<uint8_t>(b));
}
inline Permission operator~(Permission a) {
    return static_cast<Permission>(~static_cast<uint8_t>(a));
}
inline Permission& operator|=(Permission& a, Permission b) {
    a = a | b;
    return a;
}
inline Permission& operator&=(Permission& a, Permission b) {
    a = a & b;
    return a;
}

class User {
public:
    std::string name;
    Permission permissions;

    User(const std::string& name, Permission perms)
        : name(name), permissions(perms) {}

    // Check if user has ALL specified permissions
    bool hasPermission(Permission required) const {
        return (permissions & required) == required;
    }

    // Check if user has ANY of the specified permissions
    bool hasAnyPermission(Permission mask) const {
        return (permissions & mask) != Permission::None;
    }

    // Grant additional permissions
    void grant(Permission perm) {
        permissions |= perm;
    }

    // Revoke specific permissions
    void revoke(Permission perm) {
        permissions &= ~perm;
    }

    void printPermissions() const {
        uint8_t val = static_cast<uint8_t>(permissions);
        std::cout << name << ": ";
        // Print binary representation
        for (int i = 7; i >= 0; --i) {
            std::cout << ((val >> i) & 1);
        }
        std::cout << " (decimal: " << static_cast<int>(val) << ")\n";

        // Print individual permission names
        std::cout << "  Permissions: ";
        if (val == 0) { std::cout << "None"; }
        else {
            if (val & static_cast<uint8_t>(Permission::Read))    std::cout << "Read ";
            if (val & static_cast<uint8_t>(Permission::Write))   std::cout << "Write ";
            if (val & static_cast<uint8_t>(Permission::Delete))  std::cout << "Delete ";
            if (val & static_cast<uint8_t>(Permission::Create))  std::cout << "Create ";
            if (val & static_cast<uint8_t>(Permission::Admin))   std::cout << "Admin ";
            if (val & static_cast<uint8_t>(Permission::Execute)) std::cout << "Execute ";
            if (val & static_cast<uint8_t>(Permission::Share))   std::cout << "Share ";
            if (val & static_cast<uint8_t>(Permission::Audit))   std::cout << "Audit ";
        }
        std::cout << "\n";
    }
};

int main() {
    // Create users with role-based permissions
    User viewer("Alice", Permission::Viewer);
    User editor("Bob",   Permission::Editor);
    User admin("Carol",  Permission::FullAccess);

    viewer.printPermissions();
    editor.printPermissions();
    admin.printPermissions();

    // Check permissions
    std::cout << "\nBob can write: "
              << std::boolalpha << editor.hasPermission(Permission::Write)  << "\n";   // true
    std::cout << "Bob can delete: "
              << editor.hasPermission(Permission::Delete) << "\n";                     // false
    std::cout << "Bob can read AND write: "
              << editor.hasPermission(Permission::Read | Permission::Write) << "\n";   // true

    // Grant and revoke
    std::cout << "\n--- Granting Bob delete permission ---\n";
    editor.grant(Permission::Delete);
    std::cout << "Bob can delete: "
              << editor.hasPermission(Permission::Delete) << "\n";                     // true
    editor.printPermissions();

    std::cout << "\n--- Revoking Bob's write permission ---\n";
    editor.revoke(Permission::Write);
    std::cout << "Bob can write: "
              << editor.hasPermission(Permission::Write) << "\n";                      // false
    editor.printPermissions();

    return 0;
}
```

```ad-note
title: C++ enum class Requires Operator Overloading
Unlike plain `enum`, C++ `enum class` does not implicitly convert to integers, so you must overload the bitwise operators (`|`, `&`, `~`, `|=`, `&=`) yourself. This is more boilerplate but provides type safety — you cannot accidentally OR a `Permission` with an unrelated integer. In C, or with plain C++ `enum`, the operators work out of the box but offer no type safety.
```

### JavaScript — Object Constants and Bitwise Operations

JavaScript's bitwise operators work on 32-bit signed integers. This gives you up to 31 usable bit positions for flags (bit 31 is the sign bit):

```javascript
// ===== Permission Constants =====
const Permission = Object.freeze({
    NONE:    0,         // 00000000
    READ:    1 << 0,    // 00000001
    WRITE:   1 << 1,    // 00000010
    DELETE:  1 << 2,    // 00000100
    CREATE:  1 << 3,    // 00001000
    ADMIN:   1 << 4,    // 00010000
    EXECUTE: 1 << 5,    // 00100000
    SHARE:   1 << 6,    // 01000000
    AUDIT:   1 << 7,    // 10000000
});

// ===== Role Definitions =====
const Role = Object.freeze({
    VIEWER:      Permission.READ,
    EDITOR:      Permission.READ | Permission.WRITE | Permission.CREATE,
    MODERATOR:   Permission.READ | Permission.WRITE | Permission.DELETE | Permission.SHARE,
    FULL_ACCESS: Permission.READ | Permission.WRITE | Permission.DELETE | Permission.CREATE
               | Permission.ADMIN | Permission.EXECUTE | Permission.SHARE | Permission.AUDIT,
});

// ===== User Class =====
class User {
    constructor(name, permissions = Permission.NONE) {
        this.name = name;
        this.permissions = permissions;
    }

    // Check if user has ALL specified permissions
    hasPermission(required) {
        return (this.permissions & required) === required;
    }

    // Check if user has ANY of the specified permissions
    hasAnyPermission(mask) {
        return (this.permissions & mask) !== 0;
    }

    // Grant additional permissions
    grant(perm) {
        this.permissions |= perm;
    }

    // Revoke specific permissions
    revoke(perm) {
        this.permissions &= ~perm;
    }

    // Human-readable permission list
    listPermissions() {
        const names = [];
        const mapping = [
            [Permission.READ,    "Read"],
            [Permission.WRITE,   "Write"],
            [Permission.DELETE,  "Delete"],
            [Permission.CREATE,  "Create"],
            [Permission.ADMIN,   "Admin"],
            [Permission.EXECUTE, "Execute"],
            [Permission.SHARE,   "Share"],
            [Permission.AUDIT,   "Audit"],
        ];
        for (const [flag, name] of mapping) {
            if (this.permissions & flag) {
                names.push(name);
            }
        }
        return names.length > 0 ? names.join(", ") : "None";
    }

    printPermissions() {
        const binary = (this.permissions >>> 0).toString(2).padStart(8, "0");
        console.log(`${this.name}: ${binary} (decimal: ${this.permissions})`);
        console.log(`  Permissions: ${this.listPermissions()}`);
    }
}

// ===== Usage =====
const viewer = new User("Alice", Role.VIEWER);
const editor = new User("Bob",   Role.EDITOR);
const admin  = new User("Carol", Role.FULL_ACCESS);

viewer.printPermissions();
editor.printPermissions();
admin.printPermissions();

// Check permissions
console.log(`\nBob can write: ${editor.hasPermission(Permission.WRITE)}`);              // true
console.log(`Bob can delete: ${editor.hasPermission(Permission.DELETE)}`);              // false
console.log(`Bob can read AND write: ${
    editor.hasPermission(Permission.READ | Permission.WRITE)}`);                       // true

// Check ANY permission
console.log(`Alice can read OR write: ${
    viewer.hasAnyPermission(Permission.READ | Permission.WRITE)}`);                    // true

// Grant and revoke
console.log("\n--- Granting Bob delete permission ---");
editor.grant(Permission.DELETE);
console.log(`Bob can delete: ${editor.hasPermission(Permission.DELETE)}`);              // true
editor.printPermissions();

console.log("\n--- Revoking Bob's write permission ---");
editor.revoke(Permission.WRITE);
console.log(`Bob can write: ${editor.hasPermission(Permission.WRITE)}`);               // false
editor.printPermissions();
```

```ad-warning
title: JavaScript's 32-Bit Limitation
JavaScript bitwise operators convert numbers to **32-bit signed integers**. This means:
- You have 31 usable flag positions (bits 0–30). Bit 31 is the sign bit.
- `1 << 31` produces a negative number (`-2147483648`), which breaks naive comparisons.
- If you need more than 31 flags, use `BigInt` (e.g., `1n << 40n`), but be aware that `BigInt` does not mix with regular numbers in bitwise expressions.
- Use `>>> 0` (unsigned right shift by 0) to coerce to an unsigned 32-bit interpretation when printing binary.
```

---

## Comparison of Approaches Across Languages

| Feature                        | C#                         | C++                        | JavaScript                 |
|-------------------------------|:-------------------------:|:-------------------------:|:-------------------------:|
| Flag definition                | `[Flags] enum`            | `enum class` + overloads  | `Object.freeze({})` |
| Type safety                    | Strong (enum type)         | Strong (enum class)       | Weak (plain numbers)       |
| Max flags (native int)         | 32 or 64 (`long`)         | 8/16/32/64 (choose type)  | 31 (sign bit reserved)     |
| Built-in `HasFlag` method      | Yes                        | No                         | No                         |
| `ToString()` shows names       | Yes (with `[Flags]`)       | No (manual)                | No (manual)                |
| Named combinations in enum     | Yes                        | Verbose (casts needed)     | Separate object             |

```ad-tip
title: Scaling Beyond One Integer
For systems with more than 32 or 64 permissions, consider a **bitset** or **bit array** data structure rather than a single integer. C++ provides `std::bitset<N>`, C# has `BitArray`, and most languages have equivalent constructs. These support arbitrary numbers of flags while preserving the same logical operations.
```

---

## Real-World Applications

Permission bitmasks appear throughout computing:

- **File systems**: Unix permissions (9-bit), Windows ACLs (more complex but bit-flag based internally)
- **Database permissions**: MySQL `GRANT` privileges use bitmasks internally
- **API authorization**: OAuth scopes can be encoded as bit flags for fast checking
- **Game development**: Entity capabilities, feature flags, collision layers
- **Hardware registers**: CPU status flags, I/O control registers ([[Bits Bytes and Words]])
- **Feature toggles**: Enable/disable application features per user or environment
- **Network protocols**: TCP flags (SYN, ACK, FIN, RST — each a single bit in the TCP header)

```ad-note
title: Performance Advantage
Bitmask permission checks are among the fastest operations a CPU can perform. A single AND instruction (one CPU cycle) replaces what would otherwise be a database query, hash table lookup, or array search. For hot paths — middleware that checks permissions on every request, game loops checking entity states 60+ times per second — this performance difference is significant.
```

---

## Common Pitfalls and Best Practices

### Pitfalls to Avoid

1. **Using sequential values instead of powers of 2** — This is the number one mistake. Value `3` is *not* a valid flag; it is `1 | 2`.

2. **Forgetting that `0` matches the ALL check** — `(perms & 0) == 0` is always true. Never use `0` as a "required" permission in an ALL check; it will always pass.

3. **Checking `!= 0` when you need `== mask`** — The ANY vs ALL distinction (see above).

4. **Exceeding the integer width** — Shifting beyond the bit width of your integer type is undefined behavior in C/C++ and wraps in JavaScript. If you define `1 << 32` in a 32-bit int, you get undefined behavior, not your expected flag.

5. **Mutating shared constants** — In JavaScript, always `Object.freeze()` your permission and role objects. Without it, any code can accidentally write `Permission.READ = 999`.

### Best Practices

1. **Always define a `NONE = 0` value** — Makes the code self-documenting and simplifies initialization.

2. **Use `1 << n` syntax** — More readable and less error-prone than writing raw numbers. You can see the bit position at a glance.

3. **Build roles from base permissions** — Do not assign arbitrary numbers to roles. Compose them from atomic permissions using OR. This guarantees consistency.

4. **Document the bit layout** — Maintain a table or diagram of which bit means what. Future maintainers will thank you.

5. **Reserve bits for future expansion** — Do not pack your flags tightly from bit 0 upward with no room to grow. If possible, group related permissions into ranges (e.g., bits 0–3 for content, bits 4–7 for admin).

---

## Related Topics

- [[Flag Enums and Bit Flags]] — the C# `[Flags]` pattern in depth
- [[Creating Bit Masks]] — how to construct masks for isolating and combining bits
- [[AND Operator]] — used for permission checking
- [[OR Operator]] — used for granting and combining permissions
- [[NOT Operator]] — used for revoking permissions (AND NOT pattern)
- [[Check Set and Clear Bits]] — the fundamental set/clear/toggle operations
- [[Hexadecimal and Octal]] — octal notation for Unix permissions
- [[Bits Bytes and Words]] — understanding bit widths and integer sizes
- [[Binary Number System]] — how binary representation underpins all flag systems
- [[Combining and Extracting with Masks]] — mask patterns for isolating bit fields
