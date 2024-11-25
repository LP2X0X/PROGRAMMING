- An **lvalue** (pronounced “ell-value”, short for “left value” or “locator value”, and sometimes written as “l-value”) is an expression that evaluates to an identifiable object or function (or bit-field).
- Entities with identities can be accessed via an identifier, reference, or pointer, and typically have a lifetime longer than a single expression or statement.
- lvalues come in two subtypes:
	- A **modifiable lvalue** is an lvalue whose value can be modified.
	- A **non-modifiable lvalue** is an lvalue whose value can’t be modified (because the lvalue is const or constexpr).

---

- An **rvalue** (pronounced “arr-value”, short for “right value”, and sometimes written as `r-value`) is an expression that is not an lvalue. Rvalue expressions evaluate to a value, including literals and temporary objects that do not persist beyond the end of the expression. Commonly seen rvalues include literals (except C-style string literals, which are lvalues) and the return value of functions and operators that return by value. Rvalues aren’t identifiable (meaning they have to be used immediately), and only exist within the scope of the expression in which they are used.