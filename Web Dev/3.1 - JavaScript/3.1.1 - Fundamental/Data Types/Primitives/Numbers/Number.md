---
tags:
  - js
  - datatype
  - fundamental
---

- The _number_ type represents both integer and floating point numbers.
	```javascript
	alert( Infinity ); // Infinity
	```
- Besides regular numbers, there are so-called “special numeric values” which also belong to this data type: `Infinity`, `-Infinity` and `NaN`.
	- `Infinity` represents the mathematical [Infinity](https://en.wikipedia.org/wiki/Infinity) ∞. It is a special value that’s greater than any number.
		```js
		alert( Infinity ); // Infinity
		```
		
	- `NaN` represents a computational error. It is a result of an incorrect or an undefined mathematical operation, for instance:
		```javascript
		alert( "not a number" / 2 ); // NaN, such division is erroneous
		```

```ad-note
In JavaScript, the “number” type cannot safely and precisely represent integer values larger than (2<sup>53</sup>-1) (that’s `9007199254740991`), or less than -(2<sup>53</sup>-1) for negatives.
To be really precise, the “number” type can store larger integers (up to 1.7976931348623157 * 10<sup>308</sup>), but outside of the safe integer range ±(2<sup>53</sup>-1) there’ll be a precision error, because not all digits fit into the fixed 64-bit storage. So an “approximate” value may be stored.
```

```ad-tip
Evade equality checks when working with decimal fractions.
```

### More ways to write number

- We can use underscore _ as the separator. The JavaScript engine simply ignores _ between digits.
	```js
	let billion = 1_000_000_000;
	```

- We also can use "e" to shorten a numer by specifying the zeros count.
	```js
	let billion = 1e9;  // 1 billion, literally: 1 and 9 zeroes
	
	alert( 7.3e9 );  // 7.3 billions (same as 7300000000 or 7_300_000_000)
	```

- JavaScript do support other bases: hexadecimal `0xFF`, binary `0b1010`, octal `0o755`.

### Converting & Representing Numbers

- `num.toString(base)` converts numbers to different bases (2–36). 

---

### Rounding Methods

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

---

### Precision & Special Values

- Floating-point math can be imprecise (e.g. `0.1 + 0.2 !== 0.3`).

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