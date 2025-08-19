---
tags:
  - cpp
  - term
  - fundamental
---

- A **constant expression** is an [[C++/1 - Basic C++/Terms/Expressions/Expression|expression]] that contains only compile-time constants and operators/functions that support compile-time evaluation (which means it must be evaluatable at compile-time).
- Most commonly, constant expressions contain the following:
	- Literals (e.g. ‘5’, ‘1.2’)
	- Most operators with constant expression operands (e.g. `3 + 4`, `2 * sizeof(int)`).
	- **Const [[integer and integral|integral]]** variables with a constant expression initializer (e.g. `const int x { 5 };`). This is a historical exception -- in modern C++, constexpr variables are preferred.
	- Constexpr variables.
	- Constexpr function calls with constant expression arguments.

- Constant expressions can also contain:
	- Non-type template parameters.
	- Enumerators.
	- Type traits.
	- Constexpr lambda expressions.

- Notably, the following cannot be used in a constant expression:
	- Non-const variables.
	- Const non-integral variables, even when they have a constant expression initializer (e.g. `const double d { 1.2 };`). To use such variables in a constant expression, define them as constexpr variables instead.
	- The return values of non-constexpr functions (even when the return expression is a constant expression).
	- Function parameters (even when the function is constexpr).
	- Operators with operands that are not constant expressions (e.g. `x + y` when `x` or `y` is not a constant expression, or `std::cout << "hello\n"` as `std::cout` is not a constant expression).
	- Operators `new`, `delete`, `throw`, `typeid`, and `operator,` (comma).

---

- A **compile-time constant** is a constant whose value _must be_ known at compile time. This includes:
	- Literals (e.g. ‘5’, ‘1.2’)
	- [[Constexpr variable]]
	- **Const integral variables** with a constant expression initializer (e.g. `const int x { 5 };`). This is a historical exception -- in modern C++, constexpr variables are preferred.
	- Non-type template parameters
	- Enumerators
- Const variables that are not compile-time constants are sometimes called **runtime constants**. Runtime constants cannot be used in a constant expression.

```ad-tip
Const non-integral variables are always runtime constants (even if they have a constant expression initializer). If you need such variables to be compile-time constants, define them as [[Constexpr variables|constexpr variables]] instead.
```