---
tags: js, term, fundamental
---

- A **function expression**, also known as a *function literal*, is a code literal whose value is a function.
- Whereas a [[Web Dev/3.1 - JavaScript/3.1.1 - Fundamental/Functions/Function#**🛠️ Function Declaration**|function declaration]] creates a function and binds it to a name, a function expression is an expression that evaluates to (returns) a function, for you to do with what you will.
- A function expression looks very similar to a function declaration, with two main differences:
	- First, a function expression doesn’t have to include a name, although you can include one if you want.
	- Second, a function expression can’t be written at the start of a line of code, or JavaScript will think it’s a function declaration; there has to be some code before the function keyword.

```js
let addExpression = function (x, y) {
  return x + y;
};

addExpression(1, 2); // Here's how to call it
```

---

### 📌 **Syntax of a Function Expression**

```javascript
const myFunction = function(arg1, arg2) {
    return arg1 + arg2;
};
console.log(myFunction(5, 7)); // Output: 12
```

---

### 📚 **Types of Function Expressions**

1. **Anonymous Function Expression** (without a name):

```javascript
const add = function(a, b) {
    return a + b;
};
console.log(add(3, 4)); // Output: 7
```

1. **Named Function Expression** (with a name):

```javascript
const multiply = function multiplyNumbers(a, b) {
    return a * b;
};
console.log(multiply(3, 4)); // Output: 12
```

> ✅ **Note:** The function name `multiplyNumbers` is only available **inside** the function for recursion or debugging.

---

### 📊 **Function Expression vs. Function Declaration**

|Feature|Function Expression|Function Declaration|
|---|---|---|
|**Hoisting**|Not hoisted (can’t use before declaration)|Hoisted (can be called before declaration)|
|**Syntax**|Assigned to a variable|Uses `function` keyword directly|
|**Usage in Callbacks**|Commonly used in callbacks|Less common for inline callbacks|
|**Self-Reference**|Requires a named function expression for recursion|Can reference itself naturally|

Example of hoisting behavior:

```javascript
console.log(sayHello()); // ✅ Works
function sayHello() {
    return "Hello!";
}

console.log(sayHi()); // ❌ Error: sayHi is not defined
const sayHi = function() {
    return "Hi!";
};
```

---

### 📌 **When to Use Function Expressions**

- **Callbacks** (e.g., `setTimeout`, `Array.prototype.map`)
- **Event Listeners**
- **Encapsulation** (local scope)
- **Anonymous Operations** (temporary tasks)

Example with **callback**:

```javascript
const numbers = [1, 2, 3];
const squared = numbers.map(function(num) {
    return num * num;
});
console.log(squared); // [1, 4, 9]
```

---

### 🚀 **Modern Alternative: Arrow Functions**

Function expressions can also be written using **arrow functions**:

```javascript
const add = (a, b) => a + b;
console.log(add(5, 10)); // Output: 15
```

---

- In JavaScript, a function can be defined based on a condition. For example, the following function definition defines `myFunc` only if `num` equals `0`:
```js
let myFunc;
if (num === 0) {
  myFunc = function (theObject) {
    theObject.make = "Toyota";
  };
}
```

- In addition to defining functions as described here, you can also use the Function constructor to create functions from a string at runtime, much like eval().