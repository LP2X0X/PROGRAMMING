---
tags: 
 - js
 - value
 - special
---

- `NaN` stands for **Not-a-Number**.
    
- It’s a special value in JavaScript that represents the result of an **invalid numeric operation**.
    
- It is of type `number`:
    
    ```js
    typeof NaN; // "number"
    ```
    

---

### 🔹 How do you get `NaN`?

You’ll typically encounter `NaN` when:

1. **Invalid arithmetic with numbers and non-numbers**
    
    ```js
    0 / 0;       // NaN
    Math.sqrt(-1); // NaN
    ```
    
2. **Parsing fails**
    
    ```js
    Number("abc");   // NaN
    parseInt("xyz"); // NaN
    ```
    
3. **Operations with `undefined`**
    
    ```js
    undefined + 5; // NaN
    ```
    

---

### 🔹 Key properties of `NaN`

1. **It is not equal to itself**
    
    ```js
    NaN === NaN; // false
    ```
    
    This makes it unique in JavaScript.
    
2. **It is falsy but not equal to `false`**
    
    ```js
    Boolean(NaN); // false
    NaN == false; // false
    ```
    

---

### 🔹 How to check for `NaN`

- **`Number.isNaN()` (recommended)**  
    Strictly checks if a value is `NaN`:
    
    ```js
    Number.isNaN(NaN);      // true
    Number.isNaN("hello");  // false
    ```
    
- **`isNaN()` (legacy, looser)**  
    Coerces the argument to a number first:
    
    ```js
    isNaN("hello"); // true  (because Number("hello") => NaN)
    isNaN(NaN);     // true
    ```
    

---

### 🔹 Related utilities

- **`Number.isFinite()`** – checks if a value is a finite number (not `NaN`, `Infinity`, or `-Infinity`).
    
    ```js
    Number.isFinite(NaN);      // false
    Number.isFinite(10);       // true
    Number.isFinite(Infinity); // false
    ```
    
- **`Object.is()`** – can detect `NaN` because it treats `NaN` as equal to itself:
    
    ```js
    Object.is(NaN, NaN); // true
    ```
    

---

### 🔹 Common pitfalls

1. **String concatenation vs numeric addition**
    
    ```js
    "5" + 3; // "53"  (string concatenation, not NaN)
    "5" - 3; // 2     (string coerced to number)
    "a" - 3; // NaN
    ```
    
2. **Checking equality directly**
    
    ```js
    NaN == NaN;  // false
    NaN === NaN; // false
    ```
    

---

✅ **In short**:  
`NaN` is a special numeric value used to represent invalid number results.

- It’s still of type `number`.
    
- It doesn’t equal itself, so use `Number.isNaN()` or `Object.is()` to detect it.
    

---

Do you want me to also show you **real-world debugging strategies** (like how to catch where a `NaN` comes from in code)?