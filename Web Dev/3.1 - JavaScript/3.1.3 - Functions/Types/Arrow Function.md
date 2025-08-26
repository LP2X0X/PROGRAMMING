---
tags:
  - js
  - function
  - fundamental
---

- An **arrow function** is a more compact version of a [[Function Expression|function expression]]. They were introduced in **ES6 (ECMAScript 2015)** and are especially useful for short and anonymous functions.
- An arrow function doesn’t use the function keyword. Instead, it begins with the arguments list—in this case, (x, y)—followed by an arrow (=>) and the function body.
- 📌 Key Features:
	1. **Shorter syntax**: No need to write `function` keyword.
	2. **Implicit return**: If the function body has only one expression, the value is automatically returned (no `return` keyword needed).
	3. **No `this` binding**: Arrow functions do not have their own `this` context. They inherit `this` from the surrounding scope.
	4. **Always anonymous**: They don’t have a name by default.
	5. Arrow functions also have no [[The `arguments` variable|arguments]] variable.

---

- If the body consists of the arrow function is just a single statement there’s an even simpler syntax, called ***concise body***.

```js
const greet = (name) => `Hello, ${name}!`;
```

- If the arrow function has exactly one parameter, you can further simplify the syntax by omitting the parentheses around the parameter name:
  
```js
let squared = x => x * x;
```

---

### 📚 **Examples**

1. **Single parameter (no parentheses needed)**:
```javascript
const square = x => x * x;
console.log(square(4)); // 16
```

2. **Multiple parameters (parentheses required)**:
```javascript
const multiply = (a, b) => a * b;
console.log(multiply(3, 5)); // 15
```

3. **No parameters (empty parentheses)**:
```javascript
const greet = () => 'Hello, World!';
console.log(greet()); // 'Hello, World!'
```

4. **Multiline function (use curly braces)**:
```javascript
const sum = (a, b) => {
    const result = a + b;
    return result;
};
console.log(sum(10, 20)); // 30
```

5. **Return object**:
```javascript
(user) => ({ name: "John" })
```

```ad-note
You need to wrap the object literal in **parentheses `()`** when using arrow functions because without the parentheses, JavaScript thinks the curly braces `{}` mean a **function body**, not an object.
```

---

### 🔥 **Arrow Functions and `this`**
Arrow functions do **not** bind their own `this`. Instead, they capture the `this` value from their surrounding [[Lexical Environment|lexical scope]].

Example:

```javascript
function Timer() {
    this.seconds = 0;
    setInterval(() => {
        this.seconds++;
        console.log(this.seconds);
    }, 1000);
}

const timer = new Timer();
// Logs increasing numbers: 1, 2, 3...
```

If you used a **regular function**, `this` would be `undefined` or refer to the global object.

---

### 📌 **When NOT to use arrow functions**:
1. **Object methods**: `this` will not refer to the object.
```javascript
const person = {
    name: 'Alice',
    sayName: () => console.log(this.name)
};
person.sayName(); // undefined
```

2. **Constructors**: Arrow functions cannot be used as constructors.
```javascript
const Person = (name) => { this.name = name; }; // ❌ Error
const me = new Person('John'); // TypeError: Person is not a constructor
```

3. **Prototype methods**: Avoid using arrow functions if the method needs its own `this`.