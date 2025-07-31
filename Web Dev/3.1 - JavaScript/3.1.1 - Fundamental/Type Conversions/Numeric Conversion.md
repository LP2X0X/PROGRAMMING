---
tags: js, conversion, fundamental
---

- Numeric conversion in mathematical functions and expressions happens automatically. For example, when division `/` is applied to non-numbers:
	```js
	alert( "6" / "2" ); // 3, strings are converted to numbers
	```

- We can use the `Number(value)` function to explicitly convert a `value` to a number:
	```js
	let str = "123";
	alert(typeof str); // string
	
	let num = Number(str); // becomes a number 123
	
	alert(typeof num); // number
	```

- Explicit conversion is usually required when we read a value from a string-based source like a text form but expect a number to be entered.
- If the string is not a valid number, the result of such a conversion is `NaN`. For instance:
	```js
	let age = Number("an arbitrary string instead of a number");
	
	alert(age); // NaN, conversion failed
	```

- Numeric conversion rules:

|Value|Becomes…|
|---|---|
|`undefined`|`NaN`|
|`null`|`0`|
|`true and false`|`1` and `0`|
|`string`|Whitespaces (includes spaces, tabs `\t`, newlines `\n` etc.) from the start and end are removed. **If the remaining string is empty**, the result is `0`. Otherwise, the number is “read” from the string. An error gives `NaN`.|

- Example:
	```js
	alert( Number("   123   ") ); // 123
	alert( Number("123z") );      // NaN (error reading a number at "z")
	alert( Number(true) );        // 1
	alert( Number(false) );       // 0
	```