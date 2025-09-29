---
tags: 
 - js
 - error
 - type
---

### 1. What is it?

- A **special error type** introduced in ES2021.
    
- Represents **multiple errors at once**.
    
- Often used with **`Promise.any`**, which fails only if _all promises reject_.
    

---

### 2. Syntax

```js
new AggregateError(iterableOfErrors, message)
```

- **`iterableOfErrors`** → array (or iterable) of error objects (or values).
    
- **`message`** → optional string message.
    

---

### 3. Example with `Promise.any`

```js
let promises = [
  Promise.reject(new Error("Network down")),
  Promise.reject(new Error("Timeout")),
  Promise.reject("Unknown failure")
];

Promise.any(promises).catch(err => {
  console.log(err instanceof AggregateError); // true
  console.log(err.message); // "All promises were rejected"
  console.log(err.errors);  
  // [ Error: Network down, Error: Timeout, "Unknown failure" ]
});
```

---

### 4. Manual Example

You can also create one yourself:

```js
try {
  throw new AggregateError([
    new Error("First"),
    new Error("Second"),
  ], "Multiple things went wrong");
} catch (e) {
  console.log(e.message); // "Multiple things went wrong"
  console.log(e.errors);  // [ Error: First, Error: Second ]
}
```

---

### 5. Why Useful?

- Normally `throw` can only give you _one_ error.
    
- But some situations need to report _several failures at once_:
    
    - Multiple async tasks failed.
        
    - Validation errors on multiple fields.
        
    - Batch operations.
        

---

✅ **Summary**

- **`AggregateError`** bundles multiple errors into one.
    
- Used by **`Promise.any`** when everything rejects.
    
- Has an `.errors` property to inspect individual failures.
    