---
tags: 
 - js
 - object
 - term
---

## 🔹 What is a Property Descriptor?

A **property descriptor** is an object that describes the configuration of a property in JavaScript.  
It tells you **how a property behaves**, not just what its value is.

You can get a descriptor with:

```js
Object.getOwnPropertyDescriptor(obj, propName)
```

And define/change one with:

```js
Object.defineProperty(obj, propName, descriptor)
```

---

## 🔹 Two kinds of descriptors

There are **two types** of property descriptors:

### 1. **Data Descriptor**

Describes a property that holds a **value**.

It can have:

- **value** → the actual value of the property
    
- **writable** (flag) → `true` if the value can be changed
    
- **enumerable** (flag) → `true` if it shows up in iteration
    
- **configurable** (flag) → `true` if the descriptor can be changed or deleted
    

Example:

```js
let user = {};

Object.defineProperty(user, "name", {
  value: "Alice",
  writable: false,      // cannot change value
  enumerable: true,     // will show in for...in
  configurable: false   // cannot delete/modify later
});

console.log(user.name); // Alice
user.name = "Bob";      // silently fails in non-strict mode
console.log(user.name); // Alice
```

---

### 2. **Accessor Descriptor**

Describes a property that is defined with a **getter and/or setter**.

It can have:

- **get** → a function to get the property
    
- **set** → a function to set the property
    
- **enumerable** (flag)
    
- **configurable** (flag)
    

⚠️ Note: An accessor descriptor **cannot** have both `value/writable` and `get/set`. They are mutually exclusive.

Example:

```js
let user = {
  firstName: "Alice",
  lastName: "Smith"
};

Object.defineProperty(user, "fullName", {
  get() {
    return this.firstName + " " + this.lastName;
  },
  set(value) {
    [this.firstName, this.lastName] = value.split(" ");
  },
  enumerable: true,
  configurable: true
});

console.log(user.fullName); // Alice Smith
user.fullName = "Bob Stone";
console.log(user.firstName); // Bob
console.log(user.lastName);  // Stone
```

---

## 🔹 Defaults

When you create a normal property with `obj.prop = value`, the default descriptor is:

```js
{
  value: <the value>,
  writable: true,
  enumerable: true,
  configurable: true
}
```

When you create with `Object.defineProperty` and don’t specify flags, the defaults are:

```js
{
  value: undefined,
  writable: false,
  enumerable: false,
  configurable: false
}
```

---

## 🔹 Summary Table

|Type|Keys allowed|Example usage|
|---|---|---|
|Data|`value`, `writable`, `enumerable`, `configurable`|Regular properties|
|Accessor|`get`, `set`, `enumerable`, `configurable`|Computed properties|

---

👉 So a **descriptor = full definition object**  
👉 **flags = configurable, enumerable, writable** inside the descriptor