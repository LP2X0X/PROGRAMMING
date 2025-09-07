---
tags: 
 - js
 - accessor
 - property
---

## 1. What is an accessor property?

In JavaScript, object properties come in two main flavors:

1. **Data properties**
    
    - Contain a value (`{ name: "Alice" }`)
        
    - Normal key–value pairs
        
2. **Accessor properties**
    
    - Don’t hold a value directly
        
    - Instead, they define **getter** and/or **setter** functions that run when the property is accessed or assigned
        

---

## 2. Example

```js
let user = {
  firstName: "John",
  lastName: "Doe",

  // getter
  get fullName() {
    return this.firstName + " " + this.lastName;
  },

  // setter
  set fullName(name) {
    [this.firstName, this.lastName] = name.split(" ");
  }
};

console.log(user.fullName);  // "John Doe" (calls getter)
user.fullName = "Jane Smith"; // (calls setter)
console.log(user.firstName); // "Jane"
console.log(user.lastName);  // "Smith"
```

---

## 3. Defining accessor properties with `defineProperty`

You can also define them explicitly:

```js
let person = {};

Object.defineProperty(person, "age", {
  get() {
    return this._age || 0;
  },
  set(value) {
    if (value < 0) throw new Error("Age must be positive");
    this._age = value;
  },
  enumerable: true,
  configurable: true
});

person.age = 25;     
console.log(person.age); // 25
```

---

## 4. Property descriptors for accessors

Accessor properties don’t have a `value` or `writable` like data properties.  
Instead they have:

- `get` (function or `undefined`)
    
- `set` (function or `undefined`)
    
- `enumerable` (boolean)
    
- `configurable` (boolean)
    

Example descriptor:

```js
let descriptor = Object.getOwnPropertyDescriptor(user, "fullName");
console.log(descriptor);
/*
{
  get: f,
  set: f,
  enumerable: true,
  configurable: true
}
*/
```

---

## 5. Key differences

- Accessor properties **don’t store data**, they compute it dynamically.
    
- Useful for **encapsulation**, **validation**, **computed values**, and **side effects** when reading/writing.
    

---

👉 So in short:  
An **accessor property** is a property defined with a `get` and/or `set` function instead of a plain value.