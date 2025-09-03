---
tags:
  - js
  - note
---

- Two objects are equal only if they are the same object:
	- For instance, here `a` and `b` reference the same object, thus they are equal:
	
	```js
	let a = {};
	let b = a; // copy the reference
	
	alert( a == b ); // true, both variables reference the same object
	alert( a === b ); // true
	```
	
	- And here two independent objects are not equal, even though they look alike (both are empty):
	
	```js
	let a = {};
	let b = {}; // two independent objects
	
	alert( a == b ); // false
	```