---
tags: cpp, term, fundamental, universal
---
- The number of operands that an [[C++/1 - Basic C++/Terms/Operators/Operator|operator]] takes as input is called the operator’s **arity**. Operators in C++ come in four different arities:
	- **Unary** operators act on one operand.
		- An example of a unary operator is the `-` operator. For example, given `-5`, `operator-` takes literal operand `5` and flips its sign to produce new output value `-5`.
	- **Binary** operators act on two operands (often called _left_ and _right_, as the left operand appears on the left side of the operator, and the right operand appears on the right side of the operator).
		- An example of a binary operator is the `+` operator. For example, given `3 + 4`, `operator+` takes the left operand `3` and the right operand `4` and applies mathematical addition to produce new output value `7`. The insertion (`<<`) and extraction (`>>`) operators are binary operators, taking `std::cout` or `std::cin` on the left side, and the value to output or variable to input to on the right side.
	- **Ternary** operators act on three operands. There is only one of these in C++ (the conditional operator).
	- **Nullary** operators act on zero operands. There is also only one of these in C++ (the throw operator).