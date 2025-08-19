---
tags:
  - js
  - term
  - fundamental
---

- When JavaScript was first created, there were no classes, but it was still possible to share behaviors between objects using **prototype-based inheritance**.
- This older system, which still works today alongside the class system, relies on two mechanisms:
	 - A constructor function that creates and returns new objects. In this context, a constructor is just a regular, standalone function (not a function defined within a class), but it’s called using the new keyword.
	 - A prototype, an example object that the constructor uses as a model for the objects it creates. The newly created objects inherit methods and properties from the prototype object.

- Every JavaScript object has an internal link to another object called its **prototype**. This prototype object can also have a prototype, and so on, forming a **prototype chain**.


### **✅ Terminology**

- **\[\[Prototype]]** – Internal slot pointing to another object. (You can access it using Object.getPrototypeOf() or __proto__)
- **Object.prototype** – The top-level prototype object, which all objects ultimately inherit from unless specified otherwise.

---

## **🧱 How Inheritance Works**
  
```js
let animal = {
  eats: true
};

let rabbit = Object.create(animal); // rabbit inherits from animal
rabbit.hops = true;

console.log(rabbit.eats); // true (inherited)
console.log(rabbit.hops); // true (own property)
```

- rabbit doesn’t have the property eats.
- JS looks up the prototype chain: it finds eats in animal, so it returns true.

---


## **🛠️ Creating Prototypes: 3 Ways**


### **1. Using Object.create()**

```js
let car = {
  wheels: 4
};

let myCar = Object.create(car);
console.log(myCar.wheels); // 4
```

### **2. Using Constructor Functions**

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHello = function() {
  console.log("Hi, I'm " + this.name);
};

let alice = new Person("Alice");
alice.sayHello(); // Hi, I'm Alice
```

Here:

- alice.__proto__ === Person.prototype
- Person.prototype.__proto__ === Object.prototype

### **3. Using ES6 class Syntax (sugar over prototypes)**

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log("Hello " + this.name);
  }
}

const p = new Person("Bob");
p.greet(); // Hello Bob
```

Internally, it’s the same prototype chain:

```js
p → Person.prototype → Object.prototype → null
```

---

## **🔁 Inheriting from Another Prototype**

```js
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(`${this.name} makes a sound.`);
};

function Dog(name) {
  Animal.call(this, name);
}

Dog.prototype = Object.create(Animal.prototype); // Inherit
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
  console.log(`${this.name} barks.`);
};

let d = new Dog("Rex");
d.speak(); // Rex makes a sound. (inherited)
d.bark();  // Rex barks. (own method)
```

---

## **🧩 Under the Hood**

Let’s see how a lookup works with prototype inheritance:

```js
let obj = {
  a: 1
};

let child = Object.create(obj);
child.b = 2;

console.log(child.a); // JS looks: child.a → not found → obj.a → 1
```

- Lookup happens **at runtime**.
- **Method binding** (this) depends on the calling object, not where the method is defined.

---

## **❗ Pitfalls**

### **1. Shared State in Prototypes**

```js
function Person() {}
Person.prototype.hobbies = [];

let p1 = new Person();
let p2 = new Person();

p1.hobbies.push("reading");
console.log(p2.hobbies); // ["reading"] 😬 (shared!)
```

Always initialize mutable properties inside the constructor.

---

## **✅ Best Practices**

- Use Object.create() for clean prototype chains.
    
- Avoid modifying Object.prototype.
    
- Prefer class syntax if writing modern JS (ES6+).
    
- Use hasOwnProperty to check for direct properties:
    

```js
obj.hasOwnProperty('prop')
```

---

## **🧠 Summary Table**

|**Concept**|**Description**|
|---|---|
|\[\[Prototype]]|Internal link to parent object|
|__proto__|(Non-standard) Access to prototype|
|Object.create(proto)|Creates object with given prototype|
|Object.getPrototypeOf(obj)|Standard way to access prototype|
|Object.setPrototypeOf(obj, proto)|Set a new prototype (slow, use carefully)|
