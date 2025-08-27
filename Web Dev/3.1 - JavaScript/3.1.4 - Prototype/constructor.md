---
tags: 
 - js
 - property
 - advance
---


- The `constructor` property is a **reference to the function that created the instance’s prototype**.
    
- It exists by default on every **prototype object** created for a function.
    

---

## 🔍 Example 1: Default Behavior

```js
function Person(name) {
  this.name = name;
}

const user = new Person("Alice");

// Look at the prototype
console.log(Person.prototype.constructor === Person); // true
console.log(user.constructor === Person);             // true
```

- `Person.prototype.constructor` points back to `Person`.
    
- Any instance (`user`) inherits `.constructor` through the prototype chain.
    

---

## 🔍 Example 2: Custom Methods

```js
function Animal() {}
Animal.prototype.speak = function () {
  return "sound";
};

console.log(Animal.prototype.constructor === Animal); // true
```

Even if you add new methods to `Animal.prototype`, the `constructor` stays intact.

---

## ⚠️ Important Gotcha: Overwriting Prototype

If you completely overwrite a prototype, you lose the `constructor` reference.

```js
function Car() {}
Car.prototype = {
  drive() {
    return "vroom";
  },
};

const myCar = new Car();

console.log(myCar.constructor === Car); // false!
console.log(myCar.constructor === Object); // true (wrong)
```

👉 This happens because the new object literal you assigned has its own `constructor` (from `Object.prototype`), not `Car`.

✅ Fix:

```js
Car.prototype = {
  constructor: Car,  // restore constructor manually
  drive() {
    return "vroom";
  },
};
```

---

## 🔍 Example 3: ES6 Classes

With classes, JavaScript handles this for you:

```js
class Person {
  constructor(name) {
    this.name = name;
  }
}
const user = new Person("Alice");

console.log(user.constructor === Person); // true
console.log(Person.prototype.constructor === Person); // true
```

---

## 🛠️ Summary

- `.constructor` → points to the function that created the instance.
    
- Comes from the prototype object.
    
- Overwriting `.prototype` breaks it unless you restore it manually.
    
- With ES6 `class`, it’s maintained automatically.
    