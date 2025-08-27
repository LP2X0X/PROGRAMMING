---
tags: 
  - js
  - object
  - method
  - fundamental
  - static
---

### ✅ Definition

`Object.setPrototypeOf(obj, prototype)` **sets the prototype** (`[[Prototype]]`) of a given object to another object (or `null`).  
It lets you _manually change_ the prototype chain of an existing object.

---

### 🖋️ Syntax

```js
Object.setPrototypeOf(object, prototype)
```

- `object` → The target object you want to change.
    
- `prototype` → The new prototype (either another object or `null`).
    
- Returns the modified object.
    

---

### 💡 Examples

#### 1. Basic usage

```js
const animal = { eats: true };
const rabbit = { jumps: true };

Object.setPrototypeOf(rabbit, animal);

console.log(rabbit.eats);  // true (from animal)
console.log(Object.getPrototypeOf(rabbit) === animal); // true
```

---

#### 2. Setting prototype to `null`

```js
const obj = { a: 1 };
Object.setPrototypeOf(obj, null);

console.log(Object.getPrototypeOf(obj)); // null
console.log(obj.toString); // undefined (no Object.prototype in chain)
```

---

#### 3. Replacing prototype

```js
const cat = { meow: true };
const dog = { bark: true };

const pet = {};
Object.setPrototypeOf(pet, cat);
console.log(pet.meow); // true

Object.setPrototypeOf(pet, dog);
console.log(pet.bark); // true
console.log(pet.meow); // undefined
```

---

### ⚠️ Gotchas

1. **Performance hit** 🚨  
    Changing an object’s prototype dynamically can make engines like V8 (Chrome/Node.js) de-optimize the object.
    
    - Objects are optimized with their prototype fixed.
        
    - Changing it at runtime ruins that optimization.
        
    - ❌ Don’t use it in performance-critical code (like loops or hot paths).
        
2. Prototype must be **object or `null`**, else throws:
    
    ```js
    Object.setPrototypeOf({}, 123); // TypeError
    ```
    
3. For new objects, prefer **`Object.create(prototype)`** instead of creating and then resetting the prototype.
    

---

### 🛠️ Best Practices

- ✅ Use **`Object.create(proto)`** when making new objects with a specific prototype:
    
    ```js
    const rabbit = Object.create(animal);
    ```
    
    instead of:
    
    ```js
    const rabbit = {};
    Object.setPrototypeOf(rabbit, animal);
    ```
    
- ✅ Use `Object.setPrototypeOf` only when you _must_ dynamically rewire inheritance (rare).
    
- ❌ Don’t abuse it for class-like inheritance. Use `class` / `extends` instead.
    

---

### ✅ Comparison with Related Methods

- `Object.getPrototypeOf(obj)` → reads the prototype.
    
- `Object.setPrototypeOf(obj, proto)` → writes the prototype.
    
- `Object.create(proto)` → creates a new object with a given prototype.
    

---

⚡ In short:  
`Object.setPrototypeOf()` **mutates** an object’s prototype chain, but use it sparingly. For clean code, prefer `Object.create()` or `class`.