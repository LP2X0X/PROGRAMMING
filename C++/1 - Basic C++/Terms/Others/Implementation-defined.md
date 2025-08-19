---
tags:
  - cpp
  - term
  - fundamental
---

- In C++, **implementation-defined** means:
> The C++ standard allows compilers to make a choice **between two or more valid behaviors**, and requires that the compiler **document** which behavior it chooses.

### In Other Words
- **Defined by the compiler**, **not the language standard**.
- The behavior is **not portable** across different compilers or systems.
- But unlike _undefined behavior_, it's **well-defined for that specific compiler**.
