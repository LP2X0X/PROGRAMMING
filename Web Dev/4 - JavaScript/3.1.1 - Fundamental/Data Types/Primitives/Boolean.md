---
tags:
  - js
  - datatype
  - fundamental
---

- The boolean type has only two values: `true` and `false`.
- This type is commonly used to store yes/no values: `true` means “yes, correct”, and `false` means “no, incorrect”.
 - It is also represent by the number value 0 and 1:
	```js
	let a = 0;
	alert( Boolean(a) ); // false
	
	let b = "0";
	alert( Boolean(b) ); // true
	
	alert(a == b); // true!
	```
- The last statement is true because js convert all operands to number when doing [[Comparison of different types|comparison of different types]].