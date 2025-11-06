---
tags: 
 - js
 - prototype
 - advance
---


When you talk about **prototype chain**, you have to be clear:

- **Chain of instances (via `prototype`)**  
    → This is about objects created with `new`.
    
- **Chain of constructors themselves (via `__proto__`)**  
    → This is about functions being objects themselves.

---

## 🔹 A. **Instance Prototype Chain** (via `prototype` property)

This is the chain used when you create objects with `new`.

- Each function (constructor) has a `.prototype` property.
    
- Objects created with `new` will have their internal `[[Prototype]]` (`__proto__`) set to that constructor’s `.prototype`.
    
- This is how **instance methods/properties** are inherited.
    

### Example

```js
function Animal() {}
Animal.prototype.eat = function() {
  console.log("eating");
};

let rabbit = new Animal();

console.log(rabbit.__proto__ === Animal.prototype); // true
rabbit.eat(); // "eating"
```

📌 Chain here:

```text
rabbit → Animal.prototype → Object.prototype → null
```

---

## 🔹 B. **Constructor Prototype Chain** (via `__proto__` of functions)

Since functions are objects too, each function also has its own `__proto__`.  
This creates a second chain: the **constructor chain**, used for **static methods/properties**.

### Example

```js
class Animal {
  static category = "living being";
}

class Rabbit extends Animal {}

console.log(Rabbit.__proto__ === Animal); // true
console.log(Rabbit.category); // "living being"
```

📌 Chain here:

```text
Rabbit → Animal → Function.prototype → Object.prototype → null
```

---

# 🧩 2. Side-by-Side Comparison

|**Instance Chain**|**Constructor Chain**|
|---|---|
|Built via `prototype` property|Built via `__proto__` of function objects|
|Used for instance methods/properties|Used for static methods/properties|
|Example: `rabbit.eat()`|Example: `Rabbit.category`|
|rabbit → Animal.prototype → Object.prototype|Rabbit → Animal → Function.prototype → Object.prototype|

---

# 🧩 3. Unified Diagram

```
Constructor Chain (static):
Rabbit.__proto__ → Animal → Function.prototype → Object.prototype → null

Instance Chain:
rabbit.__proto__ → Animal.prototype → Object.prototype → null
```

Notice how the **two chains run in parallel**:

- Instances climb the `.prototype` chain.
    
- Constructors climb the `__proto__` chain.
    

---

# 🧩 4. Why Two Chains?

Because JavaScript separates **instance behavior** from **constructor behavior**:

- `.prototype` is for instances (shared methods).
    
- `__proto__` of functions is for constructors (static inheritance).
    

If these were merged, it would be ambiguous:

- Instances wouldn’t know where to look.
    
- Static inheritance wouldn’t work properly.
    

---

✅ In short:

- **Instance chain**: powered by `.prototype`.
    
- **Constructor chain**: powered by `__proto__`.
    
- They solve different problems but connect together to make class inheritance possible.
    