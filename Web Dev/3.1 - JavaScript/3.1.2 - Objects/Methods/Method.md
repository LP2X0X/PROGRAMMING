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
