---
tags: 
  - js
  - object
  - method
  - fundamental
  - static
---

### ✅ Definition

`Object.getPrototypeOf(obj)` returns the **prototype (the `[[Prototype]]`) of a given object**.  
It’s the modern, safe way to access what older code might use `obj.__proto__` for.

---

### 🖋️ Syntax

```js
Object.getPrototypeOf(object)
```

- `object` → the object whose prototype you want.
    
- Returns the prototype object, or `null` if none exists.
    

---

### 💡 Examples

```js
const animal = { eats: true };
const rabbit = Object.create(animal);

console.log(Object.getPrototypeOf(rabbit) === animal); // true
console.log(Object.getPrototypeOf(animal) === Object.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype)); // null
```

Compare with `__proto__`:

```js
console.log(rabbit.__proto__ === Object.getPrototypeOf(rabbit)); // true
```

---

### ⚠️ Gotchas

1. Works only on **objects** (primitives like numbers or strings throw unless wrapped).
    
    ```js
    Object.getPrototypeOf(42); // TypeError
    Object.getPrototypeOf(new Number(42)); // Number.prototype
    ```
    
2. Read-only → you can’t use it to change the prototype. Use `Object.setPrototypeOf()` or `Object.create()` for that.
    
3. Not the same as `.prototype` on functions → `.prototype` is used for constructing new objects, `getPrototypeOf` is for existing ones.
    

---

### 🛠️ Best Practices

- Use `Object.getPrototypeOf()` instead of `__proto__` (the latter is legacy and slower).
    
- Use it when you need to:
    
    - Debug prototype chains.
        
    - Implement custom inheritance.
        
    - Safely check whether an object comes from a certain prototype.
        

---

### ✅ Example: Safe Type Check

```js
function Person() {}
const john = new Person();

console.log(Object.getPrototypeOf(john) === Person.prototype); // true
```

---

⚡ So in short:

- `Object.getPrototypeOf(obj)` → get the prototype of an **existing object**.
    
- `Function.prototype` → defines the prototype for **new instances created with `new`**.
    