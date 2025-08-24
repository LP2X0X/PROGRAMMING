---
tags: 
 - js
 - number
 - method
---

- `Math.floor`, `ceil`, `round`, and `trunc` handle different rounding needs:
	- `Math.floor`: Rounds down: `3.1` becomes `3`, and `-1.1` becomes `-2`.
	- `Math.ceil`: Rounds up: `3.1` becomes `4`, and `-1.1` becomes `-1`.
	- `Math.round`: Rounds to the nearest integer: `3.1` becomes `3`, `3.6` becomes `4`. In the middle cases `3.5` rounds up to `4`, and `-3.5` rounds up to `-3`.
	- `Math.trunc` (not supported by Internet Explorer): Removes anything after the decimal point without rounding: `3.1` becomes `3`, `-1.1` becomes `-1`.

- For decimal places, use `toFixed(n)` (returns a string), or multiply → round → divide. 
	```js
	let num = 12.34;
	alert( num.toFixed(1) ); // "12.3"
	```

```ad-note
When using toFix(), the number is rounded if necessary, and the fractional part is padded with zeros if necessary so that it has the specified length.
```
