---
tags: js, operator, fundamental
---

- In classical programming, AND returns `true` if both operands are truthy and `false` otherwise.
- But in js, the AND `&&` operator does the following:
	- Evaluates operands from left to right.
	- For each operand, converts it to a boolean. If the result is `false`, stops and returns the original value of that operand.
	- If all operands have been evaluated (i.e. all were truthy), returns the last operand. 

```ad-note
In other words, AND returns the first falsy value or the last value if none were found.
```

```ad-note
Precedence of AND `&&` is higher than OR `||`
```