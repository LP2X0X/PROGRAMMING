### Binding Pattern
- The pattern starts with a declaration keyword (`var`, `let`, or `const`). Then, each individual property must either be bound to a variable or further destructured.

```js
const obj = { a: 1, b: { c: 2 } };
const {
  a,
  b: { c: d },
} = obj;
// Two variables are bound: `a` and `d`
```

```ad-note
If you want some variables to be re-assignable but others to be read-only, you may have to destructure twice — once with `let`, once with `const`.
```

- In many other syntaxes where the language binds a variable for you, you can use a binding destructuring pattern. These include:
	- The looping variable of [`for...in`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) [`for...of`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of), and [`for await...of`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of) loops;
	- [Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions) parameters;
	- The [`catch`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch) binding variable.

---

### Examples

***1. Destructuring in for...of Loops***

You can destructure elements of an iterable (like an array) when looping through it.

✅ **Example:**

```
const fruits = [
  { name: "apple", color: "red" },
  { name: "banana", color: "yellow" },
];

for (const { name, color } of fruits) {
  console.log(`${name} is ${color}`);
}
```

**Output:**

```
apple is red
banana is yellow
```

✅ **Explanation:**

• Each element of the fruits array is an object.

• { name, color } destructures these properties directly in the loop.


***2. Destructuring in for...in Loops***

You can destructure properties of an object when iterating over its keys.

✅ **Example:**

```
const user = { name: "Alice", age: 30 };

for (const key in user) {
  console.log(`${key}: ${user[key]}`);
}
```

**Output:**

```
name: Alice
age: 30
```

✅ **Explanation:**

• for...in loops over object property names (keys).

• We access the value using user[key].


***3. Destructuring in Function Parameters***
  

You can destructure function arguments directly in the parameter list.

  

✅ **Example:**

```
function printUser({ name, age }) {
  console.log(`Name: ${name}, Age: ${age}`);
}

printUser({ name: "Bob", age: 25 });
```

**Output:**

```
Name: Bob, Age: 25
```

✅ **Explanation:**

• The object passed to printUser is destructured directly in the parameter list.

• No need to manually access user.name and user.age.


***4. Destructuring in catch Binding***

When handling errors with try...catch, you can destructure properties from the error object.

  

✅ **Example:**

```
try {
  throw { message: "Something went wrong", code: 500 };
} catch ({ message, code }) {
  console.log(`Error ${code}: ${message}`);
}
```

**Output:**

```
Error 500: Something went wrong
```

✅ **Explanation:**

• The catch block receives an error object.

• We destructure the message and code properties directly.


***📌 Summary:***

Destructuring works seamlessly in:

1. **for...of loops** – Iterate over values from arrays or other iterables.

2. **for...in loops** – Iterate over object keys.

3. **Function parameters** – Destructure arguments directly.

4. **catch bindings** – Extract properties from errors.

This makes your code cleaner and reduces boilerplate!