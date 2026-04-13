---
tags:
  - cpp
  - term
  - fundamental
---

- A **temporary object** (also sometimes called an **anonymous object**) is an unnamed object that is created by the compiler to store a value temporarily.

```ad-note
Temporary objects are destroyed at the end of the [[Full Expression|full expression]] in which they are created.
```

- In modern C++ (especially since C++17), the compiler has many tricks to avoid generating temporaries where previously it would have needed to.