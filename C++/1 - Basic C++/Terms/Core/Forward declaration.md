---
tags:
  - cpp
  - term
  - fundamental
---

- A **forward declaration** allows us to tell the compiler about the existence of an identifier before actually <u>defining</u> the identifier ([[Declare vs Define]]).

```ad-note
This way, when the compiler encounters a call to the function, it’ll understand that we’re making a function call, **and can check to ensure we’re calling the function correctly**, even if it doesn’t yet know how or where the function is defined.
```

- To write a forward declaration for a function, we use a **function declaration** statement (also called a **function prototype**).

```cpp
int add(int x, int y); // function declaration includes return type, name, parameters, and semicolon.  No function body!
```

---

#### Some reasons why we need forward declaration
- Most often, forward declarations are used to tell the compiler about the existence of some function **that has been defined in a different code file**.
- Forward declarations can also be used to define our functions in an order-agnostic manner. This allows us to define functions in whatever order maximizes organization (e.g. by clustering related functions together) or reader understanding.
- Less often, there are times when we have two functions that call each other. Reordering isn’t possible in this case either, as there is no way to reorder the functions such that each is before the other. Forward declarations give us a way to resolve such circular dependencies.