---
tags: 
 - js
 - function
 - property
 - fundamental
---
- Represents the number of **declared parameters** in the function definition.
- Does **not** count rest parameters (`...args`) or default values after the first default.
```js
function foo(a, b, c) {}
console.log(foo.length); // 3

function bar(a, b = 1, c) {}
console.log(bar.length); // 1 (only `a` counted)

function baz(...args) {}
console.log(baz.length); // 0
```

⚠️ **Gotcha:** `length` is **not** about arguments passed at runtime — it’s about the function _signature_.