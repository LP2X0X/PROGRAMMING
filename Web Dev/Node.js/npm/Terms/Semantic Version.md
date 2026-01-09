---
tags: 
 - npm
 - term
---

**Semantic Versioning (SemVer)** is a versioning scheme that communicates **the impact of changes** in a software release through a structured version number.

---

## Version Format

```
MAJOR.MINOR.PATCH
```

Example:

```
2.4.1
```

---

## Meaning of Each Part

### 1. MAJOR

Increment when you make **breaking changes**.

- Backward-incompatible API changes
    
- Removing or changing existing behavior
    

Example:

```text
1.9.0 → 2.0.0
```

---

### 2. MINOR

Increment when you add **new features** in a backward-compatible way.

- New functionality
    
- Existing APIs still work
    

Example:

```text
1.4.2 → 1.5.0
```

---

### 3. PATCH

Increment when you make **backward-compatible bug fixes**.

- Bug fixes
    
- No new features
    

Example:

```text
1.5.3 → 1.5.4
```

---

## Pre-release Versions

Used for unstable releases:

```
1.0.0-alpha
1.0.0-beta.2
1.0.0-rc.1
```

Rules:

- Lower precedence than the normal release
    
- Not production-ready
    

---

## Build Metadata

Ignored for version precedence:

```
1.0.0+20260108
1.0.0+sha.abc123
```

---

## Why SemVer Matters

- Communicates risk of upgrading
    
- Enables dependency resolution (npm, Cargo, etc.)
    
- Allows automated tooling to reason about compatibility
    

---

## SemVer in npm (Important)

Version ranges:

| Range    | Meaning                     |
| -------- | --------------------------- |
| `^1.2.3` | Allow MINOR & PATCH updates | 
| `~1.2.3` | Allow PATCH updates only    |
| `1.2.3`  | Exact version               |

---

## Special Case: 0.x.y Versions

Before `1.0.0`:

- **Anything may break**
    
- MINOR may include breaking changes
    

Example:

```
0.5.0 → 0.6.0 (may break)
```

---

## One-Line Summary

> **Semantic Versioning encodes breaking changes, features, and fixes into `MAJOR.MINOR.PATCH` so users can safely reason about upgrades.**

---

## Reference
https://semver.org/