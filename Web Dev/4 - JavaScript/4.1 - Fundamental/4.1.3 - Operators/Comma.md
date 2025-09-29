---
tags:
  - js
  - operator
  - fundamental
---

- The **comma operator** lets you **evaluate multiple expressions** within a single statement.
    
- It runs each expression **from left to right**.
    
- The value of the **last expression** becomes the result of the whole comma expression.
    

Syntax:

```js
expr1, expr2, expr3, ..., exprN
```

---

### 🔹 Simple example

```js
let x = (1 + 2, 3 + 4);
console.log(x); // 7
```

- `1 + 2` is evaluated → result `3` (discarded).
    
- `3 + 4` is evaluated → result `7` (returned).
    
- `x` ends up as `7`.
    

---

### 🔹 Use cases

1. **Multiple actions in a single expression**
    
    ```js
    let a = 1;
    let b = 2;
    let result = (a += 1, b += 2, a + b);
    
    console.log(result); // 6
    // a = 2, b = 4 → a + b = 6
    ```
    
2. **In `for` loops (common usage)**  
    The comma operator is often used to update multiple variables:
    
    ```js
    for (let i = 0, j = 10; i < j; i++, j--) {
      console.log(i, j);
    }
    ```
    
    - Both `i` and `j` get updated in the loop’s `update` section.
        
3. **Compact code with side effects**
    
    ```js
    let x = 0;
    let y = (x++, x * 2);
    console.log(x, y); // 1 2
    ```
    
    - `x++` increments `x` (side effect).
        
    - `x * 2` is the final result.
        

---

### 🔹 Things to watch out for

- **Low readability**: Overusing the comma operator makes code harder to understand.
    
- **Operator precedence**: Comma has **the lowest precedence**. If you mix it with `+`, `=`, `?:`, etc., always use parentheses to avoid surprises:
    
    ```js
    let x = 1, y = 2;
    let z = (x++, y++);
    console.log(z); // 2 (not 1)
    ```
    

---

### ✅ In short

- The **comma operator** runs multiple expressions but **only returns the last one**.
    
- Mostly used in `for` loops or when you need compact code with side effects.
    
- Powerful, but should be used sparingly for readability.
    