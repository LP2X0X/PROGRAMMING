---
tags: js, destructuring, term, fundamental
---

In JavaScript, **"the rest"** typically refers to the **rest syntax** (`...`) used in functions, arrays, or objects to collect remaining values.

---

### ✅ 1. **Rest Parameters (Functions)**

Used to gather **remaining arguments** into an array:

```js
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

sum(1, 2, 3); // 6
```

Here, `...numbers` is the rest parameter.

---

### ✅ 2. **Rest with Arrays (Destructuring)**

```js
const [first, ...rest] = [1, 2, 3, 4];
console.log(first); // 1
console.log(rest);  // [2, 3, 4]
```

---

### ✅ 3. **Rest with Objects (Destructuring)**

```js
const { name, ...info } = { name: "Alice", age: 25, city: "Seoul" };
console.log(name); // "Alice"
console.log(info); // { age: 25, city: "Seoul" }
```

---

## ⚠️ Gotcha if there's no `let`

- This won’t work:
	```js
	let title, width, height;
	
	// error in this line
	{title, width, height} = {title: "Menu", width: 200, height: 100};
	```
- The problem is that JavaScript treats `{...}` in the main code flow (not inside another expression) as a code block. Such code blocks can be used to group statements, like this:
	```js
	{
	  // a code block
	  let message = "Hello";
	  // ...
	  alert( message );
	}
	```
- So here JavaScript assumes that we have a code block, that’s why there’s an error. We want destructuring instead.
- To show JavaScript that it’s not a code block, we can wrap the expression in parentheses `(...)`:
	```js
	let title, width, height;
	
	// okay now
	({title, width, height} = {title: "Menu", width: 200, height: 100});
	
	alert( title ); // Menu
	```