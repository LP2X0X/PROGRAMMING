---
tags: js, term, fundamental
---

- **Truthiness** is a special type of coercion that defines how non-Boolean values are treated as [[Boolean|Booleans]]. This allows logical operators like && and ! to be used on any type of value.
	- The **falsy** values include **undefined, null, the number 0, and an empty string ("")**.
	- All **nonzero numbers** and **nonempty strings** are truthy.

---

- The easiest way to check if a value is truthy or falsy is to apply two not operations to it using !!, meaning “not not,” a double negative. This works because the ! operator always returns a Boolean, regardless of the data type of the value it’s applied to. For example, say you want to verify that the number 0 is falsy. A single ! operation coerces the number to a Boolean, then inverts that Boolean, so !0 evaluates to true.

---

- When the && and || operators are applied to non-Booleans, they don’t return a true or false value. Instead, they return one of the original operands.
	- In the case of the && operator, if the first operand is truthy, the second operand is returned. If the first operand is falsy, the first operand is returned. 
	- The || operator works the opposite way. If the first operand is falsy, it returns the second operand, and if the first operand is truthy, it returns the first operand.

---

### Use cases
#### 1 -  Provide default behavior
```js
let name;
name = name || "No name provided";
name;
-> 'No name provided
```

#### 2 - Short-circuit
```js
let score = 0;
score && alert(`Your score is ${score}!`);
-> 0
```

```js
++score;
1
score && alert(`Your score is ${score}!`);
-> alert func get executed
```