---
tags: 
 - js
 - value
 - primitive
 - special
---

- `null` is a **primitive value** in JavaScript.
    
- It represents the **intentional absence of any object value**.
    
- It’s often used when you want to say _“this variable should have a value, but right now it doesn’t”_.
    

---

### 🔹 Type of `null`

```js
typeof null; // "object"
```

- This is a **long-standing bug** in JavaScript (from its early days).
    
- `null` is **not actually an object**, but `typeof` reports `"object"` for historical reasons.
    

```ad-note
You can read more about this here: https://2ality.com/2013/10/typeof-null.html
```

---

### 🔹 When is `null` used?

1. **Explicitly clear a variable**
    
    ```js
    let user = { name: "Alice" };
    user = null; // user no longer refers to the object
    ```
    
2. **Indicate missing value**
    
    ```js
    let selectedItem = null; // no item selected yet
    ```
    
3. **As a return value for empty results**
    
    ```js
    document.querySelector("not-there"); // null if no element found
    ```
    

---

### 🔹 Comparisons with `null`

- **Equality (`==`) vs strict equality (`===`)**
    
    ```js
    null == undefined;  // true   (loose equality)
    null === undefined; // false  (strict equality)
    ```
    
- **Falsy value**
    
    ```js
    Boolean(null); // false
    ```
    

---

### 🔹 Difference from `undefined`

- **`null`** → _“intentional absence of a value”_ (set by the programmer).
    
- **`undefined`** → _“not assigned yet”_ (default for uninitialized variables).
    

Example:

```js
let a;      // undefined
let b = null; // null

console.log(a); // undefined
console.log(b); // null
```

---

### 🔹 Use cases summary

- Use `null` when you want to **explicitly say “empty”**.
    
- Don’t confuse it with `undefined`, which usually means _uninitialized or missing_.
    

---

✅ **In short**:  
`null` is a primitive value representing _“nothing here on purpose”_. It’s falsy, has type `"object"` (due to a historical bug), and is different from `undefined`.