- In programming, a **constant** is a value that may not be changed during the program’s execution.
- C++ supports two different kinds of constants:
	- **Named constants** are constant values that are associated with an identifier. These are also sometimes called **symbolic constants**, or occasionally just **constants**.
	- **Literal constants** are constant values that are not associated with an identifier.

---

### Types of named constants
- There are three ways to define a named constant in C++:
	- Constant variables
	- Object-like macros with substitution text
	- Enumerated constants

---

### Declaring a const variable
- To declare a constant variable, we place the `const` keyword (called a “const qualifier”) adjacent to the object’s type:
```cpp
const double gravity { 9.8 };  // preferred use of const before type
int const sidesInSquare { 4 }; // "east const" style, okay but not preferred
```

```ad-note
Const variables _must_ be initialized when you define them, and then that value can not be changed via assignment.
```

```ad-note
Function parameters and return can be made constants via the const keyword. However, we should avoid doing so.
```

---

### Prefer constant variables to preprocessor macros
- There are (at least) three major problems:
	- The biggest issue is that macros don’t follow normal C++ scoping rules. Once a macro is \#defined, all subsequent occurrences of the macro’s name in the current file will be replaced.
	- Second, it is often harder to debug code using macros. Although your source code will have the macro’s name, the compiler and debugger never see the macro because it has already been replaced before they run. Many debuggers are unable to inspect a macro’s value, and often have limited capabilities when working with macros.
	- Third, macro substitution behaves differently than everything else in C++. Inadvertent mistakes can be easily made as a result.