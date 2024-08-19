```ad-note
The compiler compiles the contents of code files sequentially.
```

### Forward declaration
- A **forward declaration** allows us to tell the compiler about the existence of an identifier before actually defining the identifier.
- To write a forward declaration for a function, we use a **function declaration** statement (also called a **function prototype**).

```cpp
int add(int x, int y); // function declaration includes return type, name, parameters, and semicolon.  No function body!
```