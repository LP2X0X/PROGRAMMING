---
tags: js, datatype, fundamental
---

- A string in JavaScript must be surrounded by quotes.

```js
let str = "Hello";
let str2 = 'Single quotes are ok too';
let phrase = `can embed another ${str}`;
```

- In JavaScript, there are 3 types of quotes.
	1. Double quotes: `"Hello"`.
	2. Single quotes: `'Hello'`.
	3. Backticks: `` `Hello` ``. This is also called a [[Template Literal|template literal]]. Another advantage of using backticks is that they allow a string to span multiple lines.

- Strings can’t be changed in JavaScript. The usual workaround is to create a whole new string and assign it to `str` instead of the old one.

---

###  Accessing characters

- To get a character at position `pos`, use square brackets `[pos]` or call the method. The first character starts from the zero position:

```js
let str = `Hello`;

// the first character
alert( str[0] ); // H
alert( str.at(0) ); // H

// the last character
alert( str[str.length - 1] ); // o
alert( str.at(-1) );
```

- As you can see, the `.at(pos)` method has a benefit of allowing negative position. If `pos` is negative, then it’s counted from the end of the string.

- The square brackets always return `undefined` for negative indexes, for instance:
```js
let str = `Hello`;

alert( str[-2] ); // undefined
alert( str.at(-2) ); // l
```

- We can also iterate over characters using `for..of`:
```js
for (let char of "Hello") {
  alert(char); // H,e,l,l,o (char becomes "H", then "e", then "l" etc)
}
```