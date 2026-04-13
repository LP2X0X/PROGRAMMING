---
tags:
  - cpp
  - term
  - fundamental
---

- Conventionally, **header files** are used to propagate a bunch of related [[Forward declaration|forward declarations]] into a code file.

```ad-tip
Source files should include their paired header.
```

![[IncludeHeader.webp|center]]

- Source files should include their paired header, this allows the compiler to catch certain kinds of errors at compile time instead of link time.