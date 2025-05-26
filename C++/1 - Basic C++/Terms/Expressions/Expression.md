---
tags: cpp, term, fundamental
---

- An **expression** is a non-empty sequence of literals, variables, operators, and function calls that **calculates a value**. The process of executing an expression is called **evaluation**, and the resulting value produced is called the **result** of the expression.

````ad-note
Expressions are always evaluated as part of statements because they cannot be compiled by themselves.
````ad-example
```cpp
int x{ 2 + 3 }; // 2 + 3 is an expression that has no semicolon -- the semicolon is at the end of the statement containing the expression
```
If you were to break this statement down into its syntax, it would look like this:

`type identifier { expression };`

_type_ could be any valid type (we chose `int`). _identifier_ could be any valid name (we chose `x`). And _expression_ could be any valid expression (we chose `2 + 3`, which uses two literals and an operator).
````
