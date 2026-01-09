---
tags: 
 - npm
 - term
---

A **prerelease version** is a version number used to indicate that a release is **not yet stable** and is intended for **testing or evaluation**, not general production use.

---

## SemVer Prerelease Format

```
MAJOR.MINOR.PATCH-prerelease
```

Examples:

```
1.0.0-alpha
1.0.0-beta.2
1.0.0-rc.1
```

---

## Common Prerelease Identifiers

| Identifier               | Meaning                     |
| ------------------------ | --------------------------- |
| `alpha`                  | Early, unstable, incomplete | 
| `beta`                   | Feature-complete, testing   |
| `rc` (release candidate) | Potential final release     |

---

## Key Rules

- Prerelease versions have **lower precedence** than the corresponding normal version
    
    ```
    1.0.0-alpha < 1.0.0
    ```
    
- Numeric identifiers are compared numerically
    
    ```
    beta.2 < beta.10
    ```
    
- Prereleases are **opt-in** in most package managers
    

---

## npm-Specific Behavior

- `npm install package` **does not install prereleases by default**
    
- You must explicitly request them:
    
    ```bash
    npm install package@beta
    npm install package@1.0.0-rc.1
    ```
    
- Version ranges (`^`, `~`) **exclude prereleases** unless explicitly included
    

---

## When to Use Prerelease Versions

- Testing breaking changes
    
- Gathering early feedback
    
- CI / nightly builds
    
- Gradual rollout before a stable release
    

---

## One-Line Summary

> **A prerelease version marks software as unstable or not final and always sorts lower than the corresponding stable release.**