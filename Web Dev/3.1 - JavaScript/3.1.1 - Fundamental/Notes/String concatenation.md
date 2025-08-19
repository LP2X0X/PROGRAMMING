---
tags:
  - js
  - note
---

- String concatenation in js work similarly to other languages:
	```js
	let s = "my" + "string";
	alert(s); // mystring
	```

---

- In **JavaScript**, **string concatenation depends on the type of the first operand** (but act like a string if its met when concatenating) — and here's what that means:
	- Here, **operators work one after another**. The first `+` sums two numbers, so it returns `4`, then the next `+` adds the string `1` to it, so it’s like `4 + '1' = '41'`.
		```js
		alert(2 + 2 + '1' ); // "41" and not "221"
		```
	- Here, the first operand is a string, the compiler treats the other two operands as strings too. The `2` gets concatenated to `'1'`, so it’s like `'1' + 2 = "12"` and `"12" + 2 = "122"`.
		```js
		alert('1' + 2 + 2); // "122" and not "14"
		```
	- **This rule only apply when used with plus operator.**

````ad-note
The binary `+` is the only operator that supports strings in such a way. 
**Other arithmetic operators work only with numbers and always convert their operands to numbers.**
```javascript
alert( 6 - '2' ); // 4, converts '2' to a number
alert( '6' / '2' ); // 3, converts both operands to numbers
```
