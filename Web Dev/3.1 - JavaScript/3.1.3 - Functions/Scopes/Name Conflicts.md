---
tags: js, fundamental
---


When two arguments or variables with the **same name** exist in the scopes of a closure, a **name conflict** occurs. In JavaScript, **more deeply nested scopes take precedence** over outer scopes. This hierarchy is known as the **scope chain**.

---

### 📚 Key Rule:

- The **innermost scope** has the **highest precedence**.
    
- The **outermost scope** (global) has the **lowest precedence**.
    
- Lookup follows this chain: **inner ➡️ outer ➡️ global**
    

---

### 🧪 Example:

```js
function outside() {
  const x = 5;

  function inside(x) {
    return x * 2;
  }

  return inside;
}

console.log(outside()(10)); // 20
```

---

### 🧠 Explanation:

- There’s a **name conflict** at `return x * 2` between:
    
    - `inside`'s parameter `x` (which is `10`)
        
    - `outside`'s variable `x` (which is `5`)
        
- Because `inside` is more nested than `outside`, its `x` **shadows** the outer one.
    
- So the value `10` is used, and the result is `20`.
    
