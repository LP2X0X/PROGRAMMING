---
tags: 
 - js
 - object
 - term
 - fundamental
---

In **JavaScript**, the term **accessor** refers to a **property that is defined by a getter, a setter, or both**.

So instead of just storing a value (like a **data property**), an **accessor property** runs a function whenever you access or assign to it.

---

## 🔹 Two Types of Properties in JavaScript

1. **Data Property**
    
    - Has a value stored directly.
        
    - Example:
        
        ```js
        let obj = { x: 10 };
        console.log(obj.x); // 10
        ```
        
2. **Accessor Property (getter/setter)**
    
    - Doesn’t hold a value itself.
        
    - Instead, it defines functions for getting and/or setting the value.
        
    - Example:
        
        ```js
        let obj = {
          _x: 10,
          get x() {
            return this._x;
          },
          set x(value) {
            this._x = value * 2; // modifies how value is stored
          }
        };
        
        console.log(obj.x); // 10 (calls getter)
        obj.x = 7;          // (calls setter)
        console.log(obj.x); // 14
        ```
        

---

## 🔹 How Accessors are Represented

When you inspect an object’s property descriptors with  
`Object.getOwnPropertyDescriptor`, you can see whether it’s a data property or an accessor property:

```js
let obj = {
  get value() { return 123; }
};

console.log(Object.getOwnPropertyDescriptor(obj, "value"));
```

Output:

```js
{
  get: [Function: get value],
  set: undefined,
  enumerable: true,
  configurable: true
}
```

Notice:

- Instead of `value` and `writable`, accessor descriptors have `get` and `set`.
    

---

## 🔹 Summary

- **Accessor property** = property defined by `get` and/or `set`.
    
- It doesn’t store a value directly — instead, logic runs when reading/writing.
    
- Useful for computed values, validation, encapsulation, and controlling how properties are accessed.
    
