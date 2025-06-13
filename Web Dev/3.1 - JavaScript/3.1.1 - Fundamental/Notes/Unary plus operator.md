---
tags: js, note
---

- When the plus operator `+` applied to a single value, doesn’t do anything to numbers. But if the operand is not a number, the unary plus converts it into a number.
	```js
	// No effect on numbers
	let x = 1;
	alert( +x ); // 1
	
	let y = -2;
	alert( +y ); // -2
	
	// Converts non-numbers
	alert( +true ); // 1
	alert( +"" );   // 0
	```

- It actually does the same thing as [[Numeric Conversion|`Number(...)`]], but is shorter.