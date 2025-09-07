---
tags: 
 - js
 - prototype
 - advance
---

# 📖 What is the Prototype Chain?

In JavaScript, objects don’t copy properties from each other. Instead, they are linked in a **chain of objects** (the prototype chain).

- Each object has an internal hidden property `[[Prototype]]` (accessible as `__proto__` in most environments).
    
- When you try to read a property, JS looks:
    
    1. On the object itself.
        
    2. If not found → on `[[Prototype]]`.
        
    3. Keeps walking up until it hits the end (`null`).
        

This chain forms a **delegation system** → objects _delegate_ missing property lookups to their prototype.

---

# ⚙️ How It Works

### Example:

```js
const animal = { eats: true };
const rabbit = { jumps: true };

rabbit.__proto__ = animal;

console.log(rabbit.jumps); // true (own property)
console.log(rabbit.eats);  // true (delegated from animal)
```

Lookup steps:

- `rabbit.eats` → not found on `rabbit`.
    
- JS follows `__proto__` → finds `animal.eats`.
    

---

# 🔗 Chain Ends at `Object.prototype`

All “normal” objects eventually inherit from `Object.prototype` → which provides common methods:

- `.toString()`
    
- `.hasOwnProperty()`
    
- `.isPrototypeOf()`
    

```js
const obj = {};
console.log(obj.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__); // null
```

So the prototype chain always terminates at `null`.

---

# 🧑‍💻 Functions and `.prototype`

Functions are special:

- Functions are objects, so they have `__proto__`.
    
- Functions also have a `.prototype` property.
    
    - When you call `new MyFunc()`, the new object’s `__proto__` points to `MyFunc.prototype`.
        

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHi = function() {
  return `Hi, I’m ${this.name}`;
};

const john = new Person("John");
console.log(john.sayHi());           // works
console.log(john.__proto__ === Person.prototype); // true
```

---

# ⚡ Key Rules About the Prototype Chain

1. **Property Reads Traverse the Chain**
    
    - Only _reads_ walk the prototype chain.
        
    - Assignments (`obj.x = ...`) create or overwrite a property on the object itself.
        
2. **Methods Are Shared via Prototypes**
    
    - Good for memory efficiency: methods defined once on the prototype are shared by all instances.
        
3. **Objects Can Be Linked Arbitrarily**
    
    - You can set `__proto__` manually or use `Object.create()`.
        
4. **Chain is Dynamic**
    
    - If you change a prototype after objects are created, they will “see” the new properties.
        

---

# ✅ Best Practices

### 1. **Prefer `Object.create` over `__proto__`**

```js
const animal = { eats: true };
const rabbit = Object.create(animal); // sets prototype cleanly
rabbit.jumps = true;
```

Why?

- `__proto__` is legacy, slower, and discouraged.
    
- `Object.create(proto)` is the modern, safe way.
    

---

### 2. **Use Prototypes for Methods, Not Data**

```js
function Car(model) {
  this.model = model; // instance-specific
}
Car.prototype.drive = function() { // shared method
  console.log(`${this.model} driving`);
};
```

- Store **data per instance** in the constructor.
    
- Store **methods** on the prototype → avoids duplicating them in memory.
    

---

### 3. **Don’t Overwrite Built-in Prototypes**

- ❌ BAD:

```js
Array.prototype.shuffle = function() { ... };
```

- It can break libraries that rely on standard behavior.  

</br>

- ✅ Instead:

```js
class MyArray extends Array {
  shuffle() { ... }
}
```

---

### 4. **Know When to Use `class`**

Modern JS provides `class` syntax → just sugar over prototypes:

```js
class Animal {
  eats = true;
  walk() { console.log("walking"); }
}

class Rabbit extends Animal {
  jump() { console.log("jumping"); }
}

const r = new Rabbit();
r.walk(); // works via prototype chain
```

This is clearer, safer, and preferred in production code.

---

# ⚠️ Gotchas

1. **No Multiple Inheritance** → only one prototype per object.
    
2. **Circular References Break Things** → can’t set `obj.__proto__ = obj`.
    
3. **Shadowing Properties**
    
    - If you add the same property to both object and prototype, the object’s version “shadows” the prototype one.
        

```js
const proto = { greet: "hi" };
const obj = Object.create(proto);
obj.greet = "hello";
console.log(obj.greet); // "hello" (own prop shadows prototype)
```

---

# 🔮 Prototype Chain in Action

```js
console.log([]); 
// [].__proto__ === Array.prototype
// Array.prototype.__proto__ === Object.prototype
// Object.prototype.__proto__ === null
```

So looking up `[].toString` →

1. Not on `[]`.
    
2. Found on `Array.prototype`.
    
3. If not found → would go to `Object.prototype`.
    

---

# 🏆 Summary

- Every object has `[[Prototype]]` (`__proto__`), used for delegation.
    
- Functions also have `.prototype` to set `__proto__` of instances created with `new`.
    
- Chain ends at `Object.prototype → null`.
    
- Best practices:
    
    - Use `Object.create` instead of `__proto__`.
        
    - Keep methods on prototypes, data in constructors.
        
    - Avoid modifying built-in prototypes.
        
    - Prefer `class` syntax for clarity.
        