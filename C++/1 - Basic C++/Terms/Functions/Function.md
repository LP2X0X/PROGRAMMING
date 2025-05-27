---
tags: cpp, term, fundamental
---

- A function is a collection of [[Statement|statements]] that get executed sequentially. Functions are typically written to do a specific job or perform some useful action. It provides a way for us to split our program into small, modular chunks that are easier to organize, test and use.
- A function can return void (nothing) or value.
- When the return statement is executed, the function will return immediately (even for non-value return function).
- You can use the return value of a function as an argument of other function.

---
### Function syntax
```cpp
returnType functionName() // This is the function header (tells the compiler about the existence of the function)
{
    // This is the function body (tells the compiler what the function does)
}
```
- The first line is called the **function header**.
- The curly braces and statements in-between are called the **function body**.
---

```ad-warning
Nested functions are not supported.
```
