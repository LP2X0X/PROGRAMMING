---
tags: 
 - js
 - object
 - prototype
 - advance
---

- `Object.prototype` is the **top-level prototype** in JavaScript’s prototype chain.
    
- Almost every object inherits from it, unless explicitly created with `Object.create(null)`.
    
- It provides a set of **methods and properties** that all objects can use.
    
- Its `[[Prototype]]` is `null` → meaning it is the **end of the chain**.
    

![[Pasted image 20250827073940.png|center|300]]

```ad-important
Object with prototype here is a FUNCTION.
```

---

## 🔍 Properties and Methods of `Object.prototype`

### 1. **`constructor`**

- Points back to the `Object` function.
    

```js
console.log(Object.prototype.constructor === Object); // true
```

---

### 2. **`hasOwnProperty(prop)`**

- Checks if a property exists **directly on the object**, not in its prototype chain.
    

```js
const obj = { a: 1 };
console.log(obj.hasOwnProperty("a")); // true
console.log(obj.hasOwnProperty("toString")); // false
```

---

### 3. **`isPrototypeOf(obj)`**

- Checks if this object exists in another object’s prototype chain.
    

```js
function Animal() {}
const cat = new Animal();
console.log(Animal.prototype.isPrototypeOf(cat)); // true
```

---

### 4. **`propertyIsEnumerable(prop)`**

- Returns `true` if the property is enumerable and directly on the object.
    

```js
const obj = { a: 1 };
console.log(obj.propertyIsEnumerable("a")); // true
console.log(obj.propertyIsEnumerable("toString")); // false
```

---

### 5. **`toString()`**

- Returns a string representation of the object.
    

```js
const obj = {};
console.log(obj.toString()); // "[object Object]"
```

- Can be used to detect types:
    

```js
Object.prototype.toString.call([]); // "[object Array]"
Object.prototype.toString.call(null); // "[object Null]"
```

---

### 6. **`valueOf()`**

- Returns the **primitive value** of the object, if available.
    

```js
const num = new Number(42);
console.log(num.valueOf()); // 42
```

---

### 7. **`toLocaleString()`**

- Similar to `toString()`, but meant for localization.
    

```js
const num = 123456.789;
console.log(num.toLocaleString("de-DE")); // "123.456,789"
```

---

### 8. **`__proto__` (deprecated, but widely used)**

- Accesses an object’s prototype.
    

```js
const obj = {};
console.log(obj.__proto__ === Object.prototype); // true
```

⚠️ Modern way: `Object.getPrototypeOf(obj)` / `Object.setPrototypeOf(obj, proto)`.

---

## 🛠️ Key Notes

- `Object.prototype` is shared: changes affect **all objects**.
    

```js
Object.prototype.sayHello = function () {
  return "Hello!";
};
console.log({}.sayHello()); // "Hello!"
```

⚠️ Don’t do this in production (polluting the global prototype is bad practice).

---

## ⚡ Exceptions

- Objects created with `Object.create(null)` do **not** inherit from `Object.prototype`.
    

```js
const dict = Object.create(null);
console.log(dict.toString); // undefined
```

---

## 🧭 Prototype Chain Example

```js
const obj = { a: 1 };

console.log(obj.a); // 1 (own property)
console.log(obj.toString()); // from Object.prototype
console.log(Object.getPrototypeOf(obj)); // Object.prototype
console.log(Object.getPrototypeOf(Object.prototype)); // null
```

**Chain:**  
`obj → Object.prototype → null`

---

## ✅ **In summary**:

- `Object.prototype` is the **root** of most objects.
    
- Provides utility methods (`toString`, `hasOwnProperty`, etc.).
    
- Its prototype is `null`.
    
- Don’t modify it in production code.
    