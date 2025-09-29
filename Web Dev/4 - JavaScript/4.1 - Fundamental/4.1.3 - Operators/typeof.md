---
tags:
  - js
  - operator
  - fundamental
---

- The `typeof` operator returns the type of the operand. It’s useful when we want to process values of different types differently or just want to do a quick check.
- A call to `typeof x` returns **a string** with the type name:
	```js
	typeof undefined // "undefined"
	
	typeof 0 // "number"
	
	typeof 10n // "bigint"
	
	typeof true // "boolean"
	
	typeof "foo" // "string"
	
	typeof Symbol("id") // "symbol"
	
	typeof Math // "object"  (1)
	
	typeof null // "object"  (2)
	
	typeof alert // "function"  (3)
	```