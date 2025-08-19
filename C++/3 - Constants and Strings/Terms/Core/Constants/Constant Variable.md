---
tags:
  - cpp
  - term
  - fundamental
---

- A variable whose value cannot be changed after initialization is called a **constant variable**.

---

### Declaring a const variable
- To declare a constant variable, we place the `const` keyword (called a “const [[Type qualifiers|qualifier]]”) adjacent to the object’s type:
```cpp
const double gravity { 9.8 };  // preferred use of const before type
int const sidesInSquare { 4 }; // "east const" style, okay but not preferred
```

```ad-important
Const variables _must_ be initialized when you define them even if you use other variable to initialize it.
```

```ad-note
Function parameters and return can be made constants via the const keyword. However, we should avoid doing so since the parameter is just a copy that will be destroyed at the end of the function anyway and the `const` qualifier on a return type is simply ignored.
```
