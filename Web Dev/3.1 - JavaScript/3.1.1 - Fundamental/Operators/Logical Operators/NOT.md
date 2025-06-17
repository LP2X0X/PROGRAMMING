---
tags: js, operator, fundamental
---

- The boolean NOT operator is represented with an exclamation sign `!`.
- The operator accepts a single argument and does the following:
	1. Converts the operand to boolean type: `true/false`.
	2. Returns the inverse value.

---

### Usage 

- Check truthiness of a value:
	- The easiest way to check if a value is truthy or falsy is to apply two not operations to it using !!, meaning “not not,” a double negative.
	- This works because the ! operator always returns a Boolean, regardless of the data type of the value it’s applied to.
	-  For example, say you want to verify that the number 0 is falsy. A single ! operation coerces the number to a Boolean, then inverts that Boolean, so !0 evaluates to true.
```js
alert( !!"non-empty string" ); // true
alert( !!null ); // false
```