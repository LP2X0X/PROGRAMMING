---
tags:
  - js
  - destructuring
  - term
  - fundamental
---

In JavaScript, **"the rest"** typically refers to the **rest syntax** (`...`) used in functions, arrays, or objects to collect remaining values.

---

### ✅ 1. **Rest Parameters (Functions)**

Used to gather **remaining arguments** into an **array**:

```js
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

sum(1, 2, 3); // 6
```

Here, `...numbers` is the rest parameter. It **must be at the end**.

---

### ✅ 2. **Rest with Arrays (Destructuring)**

```js
const [first, ...rest] = [1, 2, 3, 4];
console.log(first); // 1
console.log(rest);  // [2, 3, 4]
```

---

### ✅ 3. **Rest with Objects (Destructuring)**

- When destructuring **objects**, the variable names on the left **must match the property keys** exactly (unless you use aliasing with `:`).
	```js
	const obj = { foo: 1, bar: 2 };
	
	// Must match keys exactly
	const { foo } = obj; // ✅ works
	const { Foo } = obj; // ❌ undefined (case-sensitive)
	
	// You can alias
	const { foo: myVar } = obj; // ✅ myVar = 1
	```

- When destructuring **arrays**, you don’t match by name — you match **by position**:
	```js
	const arr = [10, 20];
	const [a] = arr;   // a = 10
	const [, b] = arr; // b = 20
	```

- So:
	- **Objects → match by property name** (case-sensitive).
	- **Arrays → match by index position**.

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