---
tags: 
 - js
 - class
 - advance
 - note
---


### 1. `typeof`

- **Works for:** primitives (string, number, boolean, undefined, symbol, bigint, function, object).
    
- **Returns:** a string.
    
- Example:
    
    ```js
    typeof 42;          // "number"
    typeof "hi";        // "string"
    typeof null;        // "object" (quirk in JS)
    typeof {};          // "object"
    typeof (() => {});  // "function"
    ```
    

⚠️ Limitation: not good at distinguishing objects (`{}` vs `[]` vs `Date` → all are "object").

---

### 2. `{}.toString.call(value)`

```javascript
let user = {
  [Symbol.toStringTag]: "User"
};

alert( {}.toString.call(user) ); // [object User]
```

- **Works for:** primitives, built-in objects, and any object that supports `Symbol.toStringTag`.
    
- **Returns:** a string like `"[object Type]"`.
    
- Example:
    
    ```js
    Object.prototype.toString.call([]);        // "[object Array]"
    Object.prototype.toString.call(new Date);  // "[object Date]"
    Object.prototype.toString.call(null);      // "[object Null]"
    Object.prototype.toString.call(undefined); // "[object Undefined]"
    ```
    

This is more **precise** than `typeof`, because it can tell apart arrays, dates, regex, etc. That’s why the note says it’s a “more advanced `typeof`”.

---

### 3. `instanceof`

- **Works for:** objects only.
    
- **Returns:** true/false.
    
- Checks whether the object’s prototype chain contains the constructor’s `.prototype`.
    
- Example:
    
    ```js
    [] instanceof Array;      // true
    [] instanceof Object;     // true
    new Date() instanceof Date; // true
    new Date() instanceof Object; // true
    ```
    

⚠️ Limitation: doesn’t work across different iframes/windows (because they have different global constructors).

---

### 🔑 Key takeaway

- **`typeof`** → quick + simple, but coarse (best for primitives).
    
- **`{}.toString.call(...)`** → precise for distinguishing built-in objects.
    
- **`instanceof`** → best for checking inheritance relationships (e.g., class hierarchies).
    