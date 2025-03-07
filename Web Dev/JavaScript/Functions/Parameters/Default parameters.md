- In JavaScript, parameters of functions default to `undefined`. However, in some situations it might be useful to set a different default value. This is exactly what default parameters do.

```js
function multiply(a, b = 1) {
  return a * b;
}

console.log(multiply(5)); // 5
```