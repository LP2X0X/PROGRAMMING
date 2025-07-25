---
tags: cpp ,term, fundamental
---
- Inside the function that will return a value, we use a **return statement** to indicate the specific value being returned to the caller.
- The return statement consists of the `return` keyword, followed by an expression (sometimes called the **return expression**), ending with a semicolon.
- When the return statement is executed:
	- The return expression is evaluated to produce a value.
	- The value produced by the return expression **is copied back to the caller**. This copy is called the **return value** of the function. ***Return values are temporary objects that are destroyed at the end of the full expression.***
	- The function exits, and control returns to the caller.