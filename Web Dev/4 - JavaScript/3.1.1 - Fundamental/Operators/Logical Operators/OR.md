---
tags:
  - js
  - operator
  - fundamental
---

- In classical programming, the logical OR is meant to manipulate boolean values only. If any of its arguments are `true`, it returns `true`, otherwise it returns `false`.
- But in js, the OR `||` operator does the following:
	- Evaluates operands from left to right.
	- For each operand, converts it to boolean. If the result is `true`, stops and returns the original value of that operand.
	- If all operands have been evaluated (i.e. all were `false`), returns the last operand.

```ad-note
In other words, a chain of OR `||` returns the first truthy value or the last one if no truthy value is found.
```

---

### Usage
- Getting the first [[Truthiness|truthy]] value from a list of variables or expressions:
	```js
	let firstName = "";
	let lastName = "";
	let nickName = "SuperCoder";
	
	alert( firstName || lastName || nickName || "Anonymous"); // SuperCoder
	```
- Short-circuit evaluation: It means that `||` processes its arguments until the first truthy value is reached, and then the value is returned immediately, without even touching the other argument:
	```javascript
	true || alert("not printed");
	false || alert("printed");
	```