---
tags:
  - js
  - method
  - object
  - fundamental
---

In JavaScript, a **method** is simply a **function that is a property of an object**. It's how you define behavior for objects.

---

## 🧱 Basic Method Syntax

```js
const person = {
  name: "Alice",
  greet() {
    console.log(`Hello, I'm ${this.name}`); // First way
  },

  bow: function() { 
	console.log("Bowing!"); // Second way
  }
};

person.goodbye = function() { // Third way
	alert("Bye!");
};

function doSmt() {
	console.log('Smt');
}

person.doSmt = doSmt; // Fourth way

person.greet(); // Hello, I'm Alice
```

Here, greet() is a method of the person object. The keyword this refers to the object that owns the method (person in this case).

---

### **Using `this` in Methods**

**To access the object, a method can use the `this` keyword.**
The value of `this` is evaluated during the run-time, depending on the context.
The rule is simple: if `obj.f()` is called, then `this` is `obj` during the call of `f`.

```js
const user = {
  name: "John",
  sayHi() {
    console.log(this.name);
  }
};

user.sayHi(); // John

```

```ad-note
If you come from another programming language, then you are probably used to the idea of a “bound `this`”, where methods defined in an object always have `this` referencing that object.

In JavaScript `this` is “free”, its value is evaluated at call-time and does not depend on where the method was declared, but rather on what object is “before the dot”.
```

````ad-note
We can even call the function without an object at all:
```js
function sayHi() {
  alert(this);
}

sayHi(); // undefined
```
In this case `this` is `undefined` in strict mode. If we try to access `this.name`, there will be an error.

In non-strict mode the value of `this` in such case will be the _global object_ (`window` in a browser). This is a historical behavior that `"use strict"` fixes.
````

- In summary, The value of `this` is defined at run-time:
	- When a function is declared, it may use `this`, but that `this` has no value until the function is called.
	- A function can be copied between objects.
	- When a function is called in the “method” syntax: `object.method()`, the value of `this` during the call is `object`.

---

### **Arrow functions have no `this`**
- Arrow functions are special: they don’t have their “own” `this`. If we reference `this` from such a function, it’s taken from the **outer “normal” function**.
```js
let user = {
  firstName: "Ilya",
  sayHi() {
    let arrow = () => alert(this.firstName);
    arrow();
  }
};

user.sayHi(); // Ilya
```