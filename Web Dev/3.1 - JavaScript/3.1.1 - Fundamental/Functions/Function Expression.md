---
tags: js, term, fundamental
---

- A **function expression**, also known as a *function literal*, is a code literal whose value is a function.
- Whereas a [[Web Dev/3.1 - JavaScript/3.1.1 - Fundamental/Functions/Function#**🛠️ Function Declaration**|function declaration]] creates a function and binds it to a name, a function expression is an expression that evaluates to (returns) a function, for you to do with what you will.
- A function expression looks very similar to a function declaration, with two main differences:
	- First, a function expression doesn’t have to include a name, although you can include one if you want.
	- Second, a function expression can’t be written at the start of a line of code, or JavaScript will think it’s a function declaration; there has to be some code before the function keyword.

```js
let addExpression = function (x, y) {
  return x + y;
};

addExpression(1, 2); // Here's how to call it
```
