---
tags:
  - cpp
  - term
  - fundamental
---

- In C++, an **incomplete type** is a type that has been declared but not yet fully defined. You can declare pointers or references to it, but you **cannot create instances** or access members until the type is complete.

```ad-note
[[void]] is also an incomplete type.
```

---

### 🔹 Common Example

```cpp
class MyClass;  // Forward declaration — incomplete type

MyClass* ptr;   // OK: Pointer to incomplete type

MyClass obj;    // ❌ Error: Cannot declare object of incomplete type
```

---

### 🔹 When Incomplete Types Are Used

1. **Forward declarations** (to avoid circular dependencies)
    
2. **Pointers/references in headers** to reduce compilation dependencies
    
3. **Opaque handles** in API design
    

---

### 🔹 When It Becomes a Problem

If you:

- Try to access members of an incomplete type
    
- Try to use `sizeof()` on it
    
- Try to instantiate it directly
    

---

### 🔹 How to Fix It

Make sure the **full definition** is visible before use:

```cpp
#include "MyClass.h"  // Include the full definition

MyClass obj;  // Now OK
```
