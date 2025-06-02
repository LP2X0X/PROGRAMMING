---
tags: js, term, fundamental
---

### **\=== (Strict Equality Operator)**
- Compares **both value and type** without doing any type coercion.
- Both the type and the value must be the same to return true.

**Example:**
```js
5 === "5"  // false, because number !== string
0 === false // false, number !== boolean
null === undefined // false, different types
```