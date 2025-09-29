---
tags: 
 - web dev
---

- `undefined` is a **primitive value**.
    
- It means a variable has been **declared but not assigned a value yet**.
    
- It’s also the **default return value** of functions that don’t explicitly return anything.
    

---

### 🔹 Type of `undefined`

```js
typeof undefined; // "undefined"
```

---

### 🔹 When do you get `undefined`?

1. **Uninitialized variables**
    
    ```js
    let x;
    console.log(x); // undefined
    ```
    
2. **Missing function return**
    
    ```js
    function foo() {}
    console.log(foo()); // undefined
    ```
    
3. **Accessing missing object properties**
    
    ```js
    let obj = {};
    console.log(obj.key); // undefined
    ```
    
4. **Accessing missing array elements**
    
    ```js
    let arr = [1, 2];
    console.log(arr[5]); // undefined
    ```
    
5. **Function parameters not passed**
    
    ```js
    function greet(name) {
      console.log(name);
    }
    greet(); // undefined
    ```
    

---

### 🔹 Comparisons with `undefined`

- **Falsy value**
    
    ```js
    Boolean(undefined); // false
    ```
    
- **Loose equality (`==`) with `null`**
    
    ```js
    undefined == null;  // true
    undefined === null; // false
    ```
    

---

### 🔹 Difference from `null`

- **`undefined`** → system-level, _“not assigned yet”_.
    
- **`null`** → programmer-level, _“intentionally empty”_.
    

---

### 🔹 Good practices

- Don’t manually assign `undefined`. Use `null` if you want to clear a value.
    
- Use `===` (strict equality) to check for `undefined`.
    
- If checking for both `null` and `undefined`, use:
    
    ```js
    if (value == null) {
      // catches both null and undefined
    }
    ```
    

---

✅ **In short**:  
`undefined` means _“not set”_ or _“missing”_. It’s automatically assigned by JavaScript when something has no value yet.