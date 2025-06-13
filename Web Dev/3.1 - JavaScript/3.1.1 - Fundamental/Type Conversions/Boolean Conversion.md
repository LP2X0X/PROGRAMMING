---
tags: js, conversion, fundamental
---

- Boolean conversion is the simplest one. It happens in logical operations (later we’ll meet condition tests and other similar things) but can also be performed explicitly with a call to `Boolean(value)`.
- The conversion rule:
	- Values that are intuitively “empty”, like `0`, an empty string, `null`, `undefined`, and `NaN`, become `false`.
	- Other values become `true`.

- For instance:
	```js
	alert( Boolean(1) ); // true
	alert( Boolean(0) ); // false
	
	alert( Boolean("hello") ); // true
	alert( Boolean("") ); // false
	```