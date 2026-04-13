---
tags:
  - cpp
  - term
  - fundamental
---

- A **function declaration** statement (also called a **function prototype**) is a [[Forward declaration| forward declaration]] of a function.
	- The function declaration consists of the function’s return type, name, and parameter types, terminated with a semicolon. 
	- The names of the parameters can be optionally included. Also, the names of the parameters in the function definition don't match the declaration doesn’t matter, as the names in a declaration are optional (and if provided, ignored by the compiler).
	- The function body is not included in the declaration.

```cpp
int add(int x, int y); // function declaration includes return type, name, parameters, and semicolon.  No function body!
int add(int, int); // valid function declaration
```