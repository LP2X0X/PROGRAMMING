---
tags: 
 - js
 - function
 - method
---

## 🔹 What `bind` does

The `bind()` method **creates a new function object** that, when called, has its **`this` value** set to the value you provide, and optionally pre-fills arguments.  
It doesn’t call the function immediately — it just returns a new one.

```ad-important
The exotic [bound function](https://tc39.github.io/ecma262/#sec-bound-function-exotic-objects) object returned by `f.bind(...)` remembers the context (and arguments if provided) only at creation time.
```

---

## 🔹 Syntax

```js
function.bind(thisArg, arg1, arg2, ...)
```

- `thisArg`: the value that should be used as `this` inside the function.
    
- `arg1, arg2...`: optional arguments to pre-fill (partial application).
    

---

## 🔹 Example 1: Binding `this`

```js
const person = {
  name: "Alice",
  greet: function () {
    console.log("Hello, " + this.name);
  },
};

const greetFn = person.greet;
greetFn(); // ❌ "Hello, undefined" (because `this` is lost)

const boundGreet = person.greet.bind(person);
boundGreet(); // ✅ "Hello, Alice"
```

Without `bind`, when you take `greet` out of the object, `this` becomes undefined (or `window` in non-strict mode).  
`bind` fixes that by permanently tying the function to `person`.

---

## 🔹 Example 2: Partial application (pre-filling args)

```js
function multiply(a, b) {
  return a * b;
}

const double = multiply.bind(null, 2); // first arg (a) fixed as 2
console.log(double(5)); // 10
```

Here, `double` is a new function with `a = 2` already set.

---

## 🔹 Example 3: In event listeners

```js
class Button {
  constructor(label) {
    this.label = label;
  }

  click() {
    console.log("Button " + this.label + " clicked!");
  }
}

const btn = new Button("Submit");

// ❌ loses `this` when passed as callback
document.querySelector("button").addEventListener("click", btn.click);

// ✅ bind keeps `this`
document.querySelector("button").addEventListener("click", btn.click.bind(btn));
```

---

✅ In short:

- `bind` **locks in `this`** for a function.
    
- Can also **preset arguments**.
    
- Returns a **new function** instead of executing immediately.
    