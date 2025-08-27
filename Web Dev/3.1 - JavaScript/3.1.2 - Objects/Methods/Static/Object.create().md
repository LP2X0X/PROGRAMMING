---
tags: 
  - js
  - object
  - method
  - fundamental
  - static
---

### ✅ Definition

`Object.create(proto, [propertiesObject])` creates a **new object** with the specified prototype and optional property descriptors.  
It’s the recommended modern way to create an object with a specific prototype (instead of `Object.setPrototypeOf`).

---

### 🖋️ Syntax

```js
Object.create(proto, propertiesObject?)
```

- **`proto`** → The object to be the prototype of the new object (`[[Prototype]]`), or `null`.
    
- **`propertiesObject`** _(optional)_ → An object that defines property descriptors (like in `Object.defineProperties`).
    

Returns → A brand-new object with the given prototype.

---

### 💡 Examples

#### 1. Basic usage

```js
const animal = {
  eats: true,
  walk() {
    console.log("Animal walks");
  }
};

const rabbit = Object.create(animal); // rabbit.__proto__ = animal
rabbit.jumps = true;

console.log(rabbit.eats);  // true (from animal)
console.log(rabbit.jumps); // true (own property)
rabbit.walk();             // "Animal walks"
```

---

#### 2. Prototype set to `null`

```js
const dict = Object.create(null); // No Object.prototype in chain
dict.apple = "fruit";

console.log(dict.apple);    // "fruit"
console.log(dict.toString); // undefined (clean dictionary object)
```

👉 This is very useful for “map-like” objects where you don’t want to inherit from `Object.prototype`.

---

#### 3. With `propertiesObject`

```js
const person = {
  greet() {
    console.log("Hello!");
  }
};

const john = Object.create(person, {
  name: { value: "John", writable: true, enumerable: true }
});

console.log(john.name); // "John"
john.greet();           // "Hello!"
```

Here, `name` is defined with a property descriptor.

---

### ⚠️ Gotchas

1. If `proto` is not an object or `null`, it throws:
    
    ```js
    Object.create(123); // ❌ TypeError
    ```
    
2. Objects created with `Object.create(null)` **don’t have built-in methods** (`toString`, `hasOwnProperty`, etc.) because they don’t inherit from `Object.prototype`.
    
3. It’s more powerful than `{}` because you control the exact prototype.
    

---

### 🛠️ Best Practices

- ✅ Use `Object.create(proto)` instead of `Object.setPrototypeOf` for **creating new objects** with a prototype.
    
- ✅ Use `Object.create(null)` for **pure dictionaries** without prototype pollution risk.
    
- ❌ Don’t overuse it for inheritance when `class`/`extends` is cleaner.
    

---

### 🔑 Comparison

- `{}` → same as `Object.create(Object.prototype)`
    
- `Object.create(proto)` → lets you set any prototype.
    
- `Object.setPrototypeOf(obj, proto)` → modifies an existing object, less efficient.
    

---

⚡ **In short**:  
`Object.create()` is the **clean, modern, and preferred way** to build objects with a specific prototype, instead of mutating existing ones.