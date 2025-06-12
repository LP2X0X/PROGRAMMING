---
tags: js, datatype, fundamental
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