---
tags: 
 - js
 - method
 - number
---

### For regular number tests:

- `isNaN(value)` converts its argument to a number and then tests it for being `NaN`.
- `Number.isNaN(value)` checks whether its argument belongs to the `number` type, and if so, tests it for being `NaN`.
- `isFinite(value)` converts its argument to a number and then tests it for not being `NaN/Infinity/-Infinity`.
	- **Both `null` (cancel) and empty line in numeric form are `0`.**
- `Number.isFinite(value)` checks whether its argument belongs to the `number` type, and if so, tests it for not being `NaN/Infinity/-Infinity`.

---

### parseInt and parseFloat

- `parseInt` and `parseFloat` “read” a number from a string until they can’t. In case of an error, the gathered number is returned. The function `parseInt` returns an integer, whilst `parseFloat` will return a floating-point number:
	```js
	alert( parseInt('100px') ); // 100
	alert( parseFloat('12.5em') ); // 12.5
	
	alert( parseInt('12.3') ); // 12, only the integer part is returned
	alert( parseFloat('12.3.4') ); // 12.3, the second point stops the reading
	
	alert( parseInt('a123') ); // NaN, the first symbol stops the process
	```