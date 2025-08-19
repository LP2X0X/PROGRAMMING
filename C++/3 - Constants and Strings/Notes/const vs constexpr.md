---
tags:
  - cpp
  - distinguish
  - fundamental
---

### The meaning of const vs constexpr for variables
- For variables:
	- `const` means that the value of an object cannot be changed after initialization. The value of the initializer may be known at compile-time or runtime. The const object can be evaluated at runtime.
	- `constexpr` means that the object can be used in a constant expression. The value of the initializer must be known at compile-time. The constexpr object can be evaluated at runtime or compile-time.
- Constexpr variables are implicitly const. Const variables are not implicitly constexpr (except for const integral variables with a constant expression initializer).
- Although a variable can be defined as both `constexpr` and `const`, in most cases this is redundant, and we only need to use either `const` or `constexpr`.

```ad-tip
Any constant variable whose initializer is a constant expression should be declared as `constexpr`.
Any constant variable whose initializer is not a constant expression (making it a runtime constant) should be declared as `const`.
```

### Const and constexpr function parameters

Normal function calls are evaluated at runtime, with the supplied arguments being used to initialize the function’s parameters. Because the initialization of function parameters happens at runtime, this leads to two consequences:

1. `const` function parameters are treated as runtime constants (even when the supplied argument is a compile-time constant).
2. Function parameters cannot be declared as `constexpr`, since their initialization value isn’t determined until runtime.