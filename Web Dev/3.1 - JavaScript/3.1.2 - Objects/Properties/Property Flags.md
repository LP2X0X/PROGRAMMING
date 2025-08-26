---
tags: 
 - js
 - object
 - property
---

- Object properties, besides a **`value`**, have three special attributes (so-called “flags”):
	- **`writable`** – if `true`, the value can be changed, otherwise it’s read-only.
	- **`enumerable`** – if `true`, then listed in loops, otherwise not listed.
	- **`configurable`** – if `true`, the property can be deleted and these attributes can be modified, otherwise not.

```ad-note
When we create a property “the usual way”, all of them are `true`. But we also can change them anytime.
```

- We can get these with [[Object.getOwnPropertyDescriptor()]] and set these with [[Object.defineProperty()]] methods.

---


### 1. **`value`**

- The actual value stored in the property.
    
- Default: `undefined`
    

```js
let user = {};
Object.defineProperty(user, "name", { value: "John" });
console.log(user.name); // John
```

---

### 2. **`writable`**

- If **true**, the value can be changed.
    
- If **false**, the value is **read-only**.
    
- Default: `false`
    

```js
let user = {};
Object.defineProperty(user, "name", {
  value: "John",
  writable: false
});

user.name = "Pete"; // ❌ ignored (error in strict mode)
console.log(user.name); // John
```

---

### 3. **`enumerable`**

- If **true**, the property shows up in loops (`for...in`, `Object.keys()`).
    
- If **false**, it’s hidden from enumeration.
    
- Default: `false`
    

```js
let user = {};
Object.defineProperty(user, "secret", {
  value: "hidden",
  enumerable: false
});

console.log(Object.keys(user)); // [] (secret not shown)
```

---

### 4. **`configurable`**

- If **true**, the property descriptor can be changed or deleted.
    
- If **false**, you cannot redefine or delete it.
    
- Default: `false`
    

```js
let user = {};
Object.defineProperty(user, "id", {
  value: 123,
  configurable: false
});

// delete user.id; // ❌ Error
// Object.defineProperty(user, "id", { value: 456 }); // ❌ Error
```

```ad-important
Making a property non-configurable is a one-way road. We cannot change it back with `defineProperty`.
</br>
Also, set it to false **prevents changes of property flags and its deletion, while allowing to change its value.**
The **only exception modification** for a non-configurable **data property** is: **Change `writable: true` → `false`**.
````js
let user = {};

Object.defineProperty(user, "name", {
  value: "John",
  writable: true,
  configurable: false // Non-configurable
});

// ✅ Allowed: make writable false
Object.defineProperty(user, "name", { writable: false }); // Exception
```

---

### 5. **`get`**

- A function that is called when the property is read.
    
- No default, must be explicitly set.
    

```js
let user = {};
Object.defineProperty(user, "year", {
  get() {
    return new Date().getFullYear();
  }
});

console.log(user.year); // 2025
```

---

### 6. **`set`**

- A function that is called when the property is assigned a value.
    
- No default, must be explicitly set.
    

```js
let user = {};
let _age = 20;

Object.defineProperty(user, "age", {
  get() {
    return _age;
  },
  set(value) {
    if (value < 0) throw new Error("Age must be positive!");
    _age = value;
  }
});

user.age = 30;
console.log(user.age); // 30
```

---

## 🔹 Default Behavior Reminder

When you use `Object.defineProperty` **without specifying flags**, all of these default to:

```js
writable: false
enumerable: false
configurable: false
```

That’s why you often need to explicitly set them to `true`.

---

## ✅ Quick Summary Table

|Flag|Type|Default|Meaning|
|---|---|---|---|
|**`value`**|any|`undefined`|The actual stored value|
|**`writable`**|boolean|`false`|Can the value be changed?|
|**`enumerable`**|boolean|`false`|Does it appear in loops/`Object.keys`?|
|**`configurable`**|boolean|`false`|Can it be redefined/deleted?|
|**`get`**|function|`undefined`|Function to run when reading|
|**`set`**|function|`undefined`|Function to run when writing|
