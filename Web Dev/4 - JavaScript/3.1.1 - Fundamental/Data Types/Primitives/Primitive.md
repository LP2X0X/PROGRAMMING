---
tags:
  - js
  - term
  - fundamental
---

### ✅ What are primitives?

Primitives are the most basic data types in JavaScript.  
They are **not objects**, have **no methods** or properties of their own, and are **immutable** (cannot be changed directly).

JavaScript has **7 primitive types**:

1. **string**
    
2. **number**
    
3. **bigint**
    
4. **boolean**
    
5. **undefined**
    
6. **symbol**
    
7. **null**
    

---

### ✅ Key characteristics

1. **Immutable**
    
    - Example:
        
        ```js
        let str = "hello";
        str[0] = "H";  // ❌ won’t change
        console.log(str); // "hello"
        ```
        
        Primitives cannot be modified in place — operations always create **new values**.
        

---

2. **Copied by value**
    
    - Example:
        
        ```js
        let a = 10;
        let b = a;
        b = 20;
        console.log(a); // 10
        ```
        
        Copying a primitive creates a **new independent value**, not a reference.
        

---

3. **Not objects, but can act like them temporarily**
    
    - When you try to access a property or call a method on a primitive,  
        JavaScript **wraps it in a temporary object** using built-in constructors:
        
        - `String`
            
        - `Number`
            
        - `Boolean`
            
    
    Example:
    
    ```js
    let str = "hello";
    
    console.log(str.toUpperCase()); // "HELLO"
    ```
    
    Behind the scenes:
    
    - JavaScript does: `new String("hello")`
        
    - Calls `.toUpperCase()` on that temporary object
        
    - Discards the wrapper after the operation
        
    - The primitive value `"hello"` stays unchanged
        

---

### ✅ Special Notes for Each Primitive

- **string**
    
    - Behaves like an array of characters but immutable.
        
    - Wrapper: `String`.
        
- **number**
    
    - Includes integers and floats (IEEE 754 double precision).
        
    - Wrapper: `Number`.
        
    - Has methods like `.toFixed()`, `.toExponential()`.
        
- **bigint**
    
    - Arbitrarily large integers.
        
    - Not interchangeable with `number`.
        
    - No wrapper object like `BigInt.prototype` methods (just built-in functions).
        
- **boolean**
    
    - Only `true` or `false`.
        
    - Wrapper: `Boolean`.
        
- **undefined**
    
    - Means a variable was declared but not assigned.
        
    - No wrapper, no methods.
        
- **null**
    
    - Represents an intentional “empty” or “nothing” value.
        
    - No wrapper.  
        ⚠️ `typeof null === "object"` → historical bug in JS.
        
- **symbol**
    
    - Unique, immutable identifiers, often used as object keys.
        
    - Methods are on `Symbol` (not a wrapper object).
        

---

### ✅ Wrapper Object vs. Primitive

```js
let str1 = "hello";          // primitive string
let str2 = new String("hi"); // String object

console.log(typeof str1); // "string"
console.log(typeof str2); // "object"

console.log(str1 === "hello");   // true
console.log(str2 === "hi");      // false (object vs. string)
```

- Using `new String()`, `new Number()`, or `new Boolean()` creates an **object**, not a primitive → generally considered **bad practice**.
    

---

### ✅ Summary (important takeaways)

- Primitives are **simple, immutable values**.
    
- They are **not objects** but can behave like objects temporarily:
    
    - Accessing properties → JS creates a **temporary wrapper object** (String, Number, Boolean).
        
    - After use, the wrapper disappears.
        
- You almost never use the constructor wrappers (`new String`, etc.) — stick with primitive literals.
    
- Operations on primitives return **new values**, they never modify the original.
    
