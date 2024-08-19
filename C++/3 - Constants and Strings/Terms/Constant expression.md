- One kind of expression that can always be evaluated at compile time is called a **constant expression**. The precise definition of a constant expression is complicated, so we’ll take a simplified view: A **constant expression** is an expression that contains only compile-time constants and operators/functions that support compile-time evaluation.
- A **compile-time constant** is a constant whose value _must be_ known at compile time. This includes:
	- Literals (e.g. ‘5’, ‘1.2’)
	- [[Constexpr variables]]
	- Const integral variables with a constant expression initializer (e.g. `const int x { 5 };`). This is a historical exception -- in modern C++, constexpr variables are preferred.
	- Non-type template parameters
	- Enumerators
- Const variables that are not compile-time constants are sometimes called **runtime constants**. Runtime constants cannot be used in a constant expression.

```ad-tip
Const non-integral variables are always runtime constants (even if they have a constant expression initializer). If you need such variables to be compile-time constants, define them as [[Constexpr variables|constexpr variables]] instead.
```