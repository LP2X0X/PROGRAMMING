---
tags: 
 - js
 - function
 - property
 - advance
---

# 📖 What is `prototype`?

- `prototype` is a **property automatically created on functions** in JavaScript.
    
- It is an object that will be assigned as the **prototype of instances** created with `new`.
    
- This is the key mechanism that powers **prototypal inheritance** in JS.
    

```ad-note
If prototype is modified and point to something else it does not affect the prototype of existing ones.
```

---

# Example

```js
let animal = {
  eats: true
};

function Rabbit(name) {
  this.name = name;
}

Rabbit.prototype = animal;

let rabbit = new Rabbit("White Rabbit"); //  rabbit.__proto__ == animal

alert( rabbit.eats ); // true
```
- Setting `Rabbit.prototype = animal` literally states the following: “When a `new Rabbit` is created, assign its `[[Prototype]]` to `animal`”.
- That’s the resulting picture:
![[Pasted image 20250826154758.png|center|400]]

---

## 🔹 1. Where does `prototype` live?

Every function in JavaScript (except arrow functions) has a `prototype` property.

```js
function Animal() {}
console.log(typeof Animal.prototype); // "object"
```

By default:

```js
Animal.prototype = {
  constructor: Animal,   // points back to function itself
  __proto__: Object.prototype
}
```

So when you create an instance:

```js
const dog = new Animal();
```

Internally:

```js
dog.__proto__ === Animal.prototype; // true
```

---

## 🔹 2. Adding methods to `prototype`

Anything you add to `prototype` will be shared across all instances.  
This saves memory (instead of duplicating methods on each object).

```js
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  return `${this.name} makes a sound.`;
};

const dog = new Animal("Dog");
const cat = new Animal("Cat");

console.log(dog.speak()); // "Dog makes a sound."
console.log(cat.speak()); // "Cat makes a sound."
```

👉 Both `dog` and `cat` share the **same function** from `Animal.prototype`.

---

## 🔹 3. The `constructor` property

By default, `prototype` has a `.constructor` that points back to the function:

```js
console.log(Animal.prototype.constructor === Animal); // true
```

This allows you to do:

```js
const anotherDog = new dog.constructor("Another Dog");
```

⚠️ If you replace the prototype object entirely, you might lose this reference:

```js
Animal.prototype = { sound: "woof" };
console.log((new Animal()).constructor === Animal); // false (now it's Object)
```

You’d need to manually reset `constructor`.

---

## 🔹 4. Prototype Chain with Functions

Every function is itself an object, so it also has a prototype chain.

```js
function Animal() {}
console.log(Animal.prototype);  // object with constructor
console.log(Object.getPrototypeOf(Animal)); // ƒ Function
```

---

## 🔹 5. `prototype` in Classes

Modern `class` syntax is just syntactic sugar over `prototype`.

```js
class Animal {
  speak() {
    return "sound";
  }
}

console.log(typeof Animal.prototype); // "object"
console.log(Object.getOwnPropertyNames(Animal.prototype));
// ["constructor", "speak"]
```

So behind the scenes:

```js
class Animal {}
```

is almost the same as:

```js
function Animal() {}
Animal.prototype = { constructor: Animal };
```

---

## 🔹 6. Change prototype

When you do:

```js
function Rabbit() {}
const rabbit1 = new Rabbit();

Rabbit.prototype.jumps = true; // ✅ works, affects rabbit1
```

- At the time you created `rabbit1`, JS linked its internal `[[Prototype]]` to whatever `Rabbit.prototype` was at that moment.
    
- Adding or modifying properties on **that same object** (`Rabbit.prototype`) is reflected in existing instances.
    

But if you **reassign** the whole `prototype`:

```js
Rabbit.prototype = { runs: true };

const rabbit2 = new Rabbit();

console.log(rabbit1.runs); // ❌ undefined
console.log(rabbit2.runs); // ✅ true
```

- `rabbit1` still points to the **old prototype object**.
    
- `rabbit2` points to the **new prototype object**.
    

👉 So: _Changing the `Rabbit.prototype` reference only affects future instances, not existing ones._

---

## ✅ Summary of `prototype`

- Exists **only on functions**.
    
- Used to set up the **prototype for instances** created with `new`.
    
- By default, contains `{ constructor: FunctionName }`.
    
- Methods added to `.prototype` are **shared** across all instances.
    
- Forms the **prototype chain** for inheritance.
    
- Modern `class` syntax hides this, but it still uses `.prototype` under the hood.
    
