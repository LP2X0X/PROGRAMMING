---
tags:
  - cpp
  - term
  - fundamental
---

- A **full expression** is an expression that is not a [[Sub Expression|subexpression]].

#### Key points:

- It is **not contained** in another expression.
- It determines the **sequence point** (i.e., when side effects must be completed).
- Temporary objects are **destroyed** at the end of the full-expression unless lifetime is extended.

#### Examples:

1. In an assignment:
    
    ```cpp
    int x = a + b;
    ```
    
    - Full-expression: `a + b`
        
2. In a statement:
    
    ```cpp
    std::cout << "Hello\n";
    ```
    
    - Full-expression: `std::cout << "Hello\n"`
        
3. In a constructor argument:
    
    ```cpp
    MyClass obj(a + b, c * d);
    ```
    
    - `a + b` and `c * d` are sub-expressions.
    - The full-expression is the entire constructor call.
        
4. In a return statement:
    
    ```cpp
    return x + y;
    ```
    
    - `x + y` is a full-expression (because it's the last thing before returning).
        

---

### 🔹 Why It Matters

- **Sequencing of side effects**:
    - Modifications must complete by the end of a full-expression.
- **Lifetime of temporaries**:
    - Temporaries created within a full-expression are destroyed at its end (unless lifetime is extended).
