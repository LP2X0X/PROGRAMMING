---
tags: js, term, fundamental
---

- An **arrow function** is a more compact version of a [[Function Expression|function expression]].
- An arrow function doesn’t use the function keyword. Instead, it begins with the arguments list—in this case, (x, y)—followed by an arrow (=>) and the function body.
- If the body consists of the arrow function is just a single statement there’s an even simpler syntax, called ***concise body***.

```js
const greet = (name) => `Hello, ${name}!`;
```

- If the arrow function has exactly one parameter, you can further simplify the syntax by omitting the parentheses around the parameter name:
  
```js
let squared = x => x * x;
```
