---
tags: 
 - js
 - technique
---

In JavaScript you want to be safe against:

- `null`
    
- `undefined`
    
- `NaN`
    
- and non-numbers (like strings, objects, etc).
    

---

### ✅ Best way

```js
function isValidNumber(value) {
  return typeof value === "number" && Number.isFinite(value);
}
```

---

### Why this works

- `typeof value === "number"` → filters out `null`, `undefined`, strings, objects, etc.
    
- `Number.isFinite(value)` → rejects `NaN`, `Infinity`, `-Infinity`.
    

---

### Examples

```js
isValidNumber(42);        // true
isValidNumber(0);         // true
isValidNumber(-3.14);     // true

isValidNumber(null);      // false
isValidNumber(undefined); // false
isValidNumber(NaN);       // false
isValidNumber("123");     // false
isValidNumber(Infinity);  // false
```

---

⚡ If you specifically want to **allow Infinity** but just exclude `NaN`, you could instead use:

```js
function isNumber(value) {
  return typeof value === "number" && !Number.isNaN(value);
}