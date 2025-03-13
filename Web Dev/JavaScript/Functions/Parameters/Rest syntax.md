- The [rest parameter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters) syntax allows us to represent an indefinite number of arguments as an array.
- In the following example, the function `multiply` uses _rest parameters_ to collect arguments from the second one to the end. The function then multiplies these by the first argument.

```js
function multiply(multiplier, ...theArgs) {
  return theArgs.map((x) => multiplier * x);
}

const arr = multiply(2, 1, 2, 3);
console.log(arr); // [2, 4, 6]
```