---
tags: 
 - js
 - distinguish
 - advance
---

1. **Function property syntax**:
    
    ```js
    eat: function() { ... }
    ```
    
    - Creates a _function-valued property_.
        
    - No `[[HomeObject]]` → `super` won’t work.
        
    - Treated like any other plain function stored in an object.
        
2. **Method shorthand syntax**:
    
    ```js
    eat() { ... }
    ```
    
    - Creates a proper _method_.
        
    - Adds the internal `[[HomeObject]]` slot.
        
    - Enables `super` to work correctly.
        
