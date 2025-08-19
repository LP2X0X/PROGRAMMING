---
tags:
  - cpp
  - term
  - fundamental
---

- For backwards compatibility reasons, many C++ compilers support **multicharacter literals**, which are char literals that contain multiple characters (e.g. `'56'`). If supported, these have an implementation-defined value (meaning it varies depending on the compiler). 

---

### Why you should avoid them

#### 1. **Implementation-defined behavior**

- The standard allows multi-character literals, but how they are handled is **not defined** in a portable way.
    
- Different compilers may give different results for the same literal.
    

#### 2. **Platform-dependent encoding**

- On most platforms, a multi-character literal is stored as an `int` where each character is packed into the int’s bytes (e.g., `'AB'` becomes `0x4142` or `0x4241` depending on endianness).
    

#### 3. **Not intended for normal use**

- These are not meant for typical logic or meaningful values. They can lead to confusing and buggy code.
    

#### 4. **Compiler warnings or silent issues**

- Some compilers may warn you; others might not.
    
- The result may seem to “work” but break silently on another platform or architecture.
    

---
- Because they are not part of the C++ standard, and their value is not strictly defined, multicharacter literals **should be avoided**.