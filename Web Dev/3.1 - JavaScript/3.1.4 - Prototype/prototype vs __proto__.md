---
tags: 
 - js
 - distinguish
 - advance
---

### 🔹 `prototype` (property on functions)

- Exists **only on functions** (constructor functions or classes).
    
- Used as a **template object** that will become the prototype of instances created with `new`.
    

```js
function Animal(name) {
  this.name = name;
}

console.log(Animal.prototype); 
// { constructor: ƒ Animal, __proto__: Object.prototype }
```

So if you do:

```js
const dog = new Animal("Dog");
```

Then:

```js
dog.__proto__ === Animal.prototype; // true
```

---

### 🔹 `__proto__` (internal slot on objects)

- Exists on **all objects** (except ones created with `Object.create(null)`).
    
- It’s an accessor to the hidden `[[Prototype]]` internal property.
    
- Points to the object’s **parent in the prototype chain**.
    

```js
const dog = new Animal("Dog");

console.log(dog.__proto__);       
// → Animal.prototype
console.log(dog.__proto__ === Animal.prototype); 
// true
```

---

# 🧭 Key Differences

|Feature|`prototype`|`__proto__`|
|---|---|---|
|Belongs to|Functions (constructor/class)|All objects|
|Type|Regular object property|Accessor to hidden `[[Prototype]]`|
|Used for|Setting up inheritance for new instances|Looking up parent in prototype chain|
|Example|`Animal.prototype.bark = () => "woof"`|`dog.__proto__ === Animal.prototype`|

---

# ⚡ Visual Diagram

```js
function Animal() {}

const dog = new Animal();

// Relationships:
dog.__proto__ === Animal.prototype      // true
Animal.prototype.constructor === Animal // true
Object.getPrototypeOf(dog) === Animal.prototype // true (modern safe way)
```

Chain:

```
dog → Animal.prototype → Object.prototype → null
```

---

# ✅ Summary

- `prototype`: special property of functions, used **when creating objects with `new`**.
    
- `__proto__`: hidden link inside objects, pointing to their prototype (the one they inherit from).
    
- In modern code, use `Object.getPrototypeOf(obj)` instead of `obj.__proto__` (the latter is legacy but still widely supported).
    