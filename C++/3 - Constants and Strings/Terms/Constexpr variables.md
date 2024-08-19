- We can enlist the compiler’s help to ensure we get a compile-time constant variable where we desire one. To do so, we use the `constexpr` keyword (which is shorthand for “constant expression”) instead of `const` in a variable’s declaration. A **constexpr** variable is always a compile-time constant. As a result, a constexpr variable must be initialized with a constant expression, otherwise a compilation error will result.

---
### The meaning of const vs constexpr for variables
- For variables:
	- `const` means that the value of an object cannot be changed after initialization. The value of the initializer may be known at compile-time or runtime. The const object can be evaluated at runtime.
	- `constexpr` means that the object can be used in a constant expression. The value of the initializer must be known at compile-time. The constexpr object can be evaluated at runtime or compile-time.
- Constexpr variables are implicitly const. Const variables are not implicitly constexpr (except for const integral variables with a constant expression initializer).
- Although a variable can be defined as both `constexpr` and `const`, in most cases this is redundant, and we only need to use either `const` or `constexpr`.

```ad-tip
Best practice:
Any constant variable whose initializer is a constant expression should be declared as `constexpr`.
Any constant variable whose initializer is not a constant expression (making it a runtime constant) should be declared as `const`.
```