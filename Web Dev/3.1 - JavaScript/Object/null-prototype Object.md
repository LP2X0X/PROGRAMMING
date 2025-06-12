A **null prototype object** in JavaScript is an object that **does not inherit from `Object.prototype`**. This means it has no built-in methods like `toString()`, `hasOwnProperty()`, etc.

---

### 📌 **How to Create a Null Prototype Object**

You can create an object with a `null` prototype using:

1. **`Object.create(null)`** – The most common method.
2. **`Object.setPrototypeOf(obj, null)`** – To remove the prototype of an existing object.

---

### ✅ **Example 1: Creating a Null Prototype Object**

```javascript
const obj = Object.create(null);
obj.name = 'No Prototype';

console.log(obj); // { name: 'No Prototype' }
console.log(Object.getPrototypeOf(obj)); // null
```

- ✅ **No prototype chain**: Methods like `toString()` are unavailable.
- ✅ **Ideal for creating pure data objects** (e.g., dictionaries).

---

### 🔍 **Example 2: No Access to `Object.prototype` Methods**

```javascript
const obj = Object.create(null);
obj.name = 'Test';

console.log(obj.hasOwnProperty('name')); // ❌ TypeError: obj.hasOwnProperty is not a function
```

Since `obj` has no prototype, it does **not** inherit the `hasOwnProperty()` method.

✅ **Workaround:**

```javascript
console.log(Object.prototype.hasOwnProperty.call(obj, 'name')); // true
```

---

### 🛠️ **Example 3: Removing an Existing Prototype**

```javascript
const obj = { a: 1 };
Object.setPrototypeOf(obj, null);

console.log(Object.getPrototypeOf(obj)); // null
console.log(obj.toString); // undefined
```

---

### 📚 **When to Use Null Prototype Objects**

1. **Creating pure data containers**: To avoid prototype interference.
2. **Safe key storage**: When keys may conflict with object methods (e.g., `toString`).
3. **Reducing prototype pollution**: Ensures no unexpected prototype methods.

✅ **Example: Using a null prototype for a safe map-like object:**

```javascript
const map = Object.create(null);
map['__proto__'] = 'Oops!';

console.log(map.__proto__); // "Oops!" (Safe from prototype pollution)
```

Would you like to dive deeper into advanced use cases or best practices?