---
tags: js, datatype, fundamental
---

- `BigInt` type was recently added to the language to represent integers of arbitrary length. This datatype helps remove the limitation of fundamental [[Number|number]] type.
- A `BigInt` value is created by appending `n` to the end of an integer:
	```javascript
	// the "n" at the end means it's a BigInt
	const bigInt = 1234567890123456789012345678901234567890n;
	```