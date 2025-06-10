- The [rest parameter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters) syntax allows us to represent an indefinite number of arguments as an array.
- In the following example, the function `multiply` uses _rest parameters_ to collect arguments from the second one to the end. The function then multiplies these by the first argument.

```js
function multiply(multiplier, ...theArgs) {
  return theArgs.map((x) => multiplier * x);
}

const arr = multiply(2, 1, 2, 3);
console.log(arr); // [2, 4, 6]
```

---

### Rest properties and Rest elements
the `...rest` syntax is called "rest elements" in array destructuring and "rest properties" in object destructuring, but we often just collectively call them "rest property".

```js
const { a, ...others } = { a: 1, b: 2, c: 3 };
console.log(others); // { b: 2, c: 3 }

const [first, ...others2] = [1, 2, 3];
console.log(others2); // [2, 3]
```

```js
const person = {
  name: "Alice",
  age: 25,
  country: "USA",
  job: "Developer",
};

// Destructuring with rest properties
const { name, age, ...rest } = person;

console.log(name); // "Alice"
console.log(age);  // 25
console.log(rest); // { country: "USA", job: "Developer" }
```