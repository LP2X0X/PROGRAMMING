### Function declarations

- A **function definition** (also called a **function declaration**, or **function statement**) consists of the function keyword, followed by:
	- The name of the function.
	- A list of parameters to the function, enclosed in parentheses and separated by commas.
	- The JavaScript statements that define the function, enclosed in curly braces, `{ /* … */ }`.
</br>
- For example, the following code defines a function named `square`:
```js
function square(number) {
  return number * number;
```

---

### Parameters passing
- Parameters are essentially passed to functions **by value** — so if the code within the body of a function assigns a completely new value to a parameter that was passed to the function, **the change is not reflected globally or in the code which called that function**.
</br>
- When you pass an object as a parameter, if the function changes the object's properties, that change is visible outside the function.

```js
function myFunc(theObject) {
  theObject.make = "Toyota";
}

const myCar = {
  make: "Honda",
  model: "Accord",
  year: 1998,
};

console.log(myCar.make); // "Honda"
myFunc(myCar);
console.log(myCar.make); // "Toyota"
```
</br>
- When you pass an array as a parameter, if the function changes any of the array's values, that change is visible outside the function.

```js
function myFunc(theArr) {
  theArr[0] = 30;
}

const arr = [45];

console.log(arr[0]); // 45
myFunc(arr);
console.log(arr[0]); // 30
```
</br>
- Function declarations and expressions can be nested, which forms a _scope chain_.

```js
function addSquares(a, b) {
  function square(x) {
    return x * x;
  }
  return square(a) + square(b);
}
```

---

### Calling functions
- **Calling** the function actually performs the specified actions with the indicated parameters.
- Functions must be _in scope_ when they are called, but the function declaration can be [hoisted](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions#function_hoisting) (appear below the call in the code). The scope of a function declaration is the function in which it is declared (or the entire program, if it is declared at the top level).
- It turns out that _functions are themselves objects_ — and in turn, these objects have methods. (See the [`Function`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function) object.) The [`call()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call) and [`apply()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) methods can be used to achieve this goal.

```ad-note
Function hoisting only works with function _declarations_ — not with function _expressions_.
```