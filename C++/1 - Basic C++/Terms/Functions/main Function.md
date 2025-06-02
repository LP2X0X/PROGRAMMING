---
tags: cpp, fundamental
---

- Every C++ program must have a special [[C++/1 - Basic C++/Terms/Functions/Function|function]] named **main** (all lower case letters). When the program is run, the statements inside of `main` are executed in sequential order.
```ad-note
It is a common misconception that `main` is always the first function that executes.
Global variables are initialized prior to the execution of `main`. If the initializer for such a variable invokes a function, then that function will execute prior to `main`.
```

- Function main will implicitly return 0 if no return statement is provided.