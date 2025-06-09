---
tags: cpp, term, fundamental
---

- In programming, a **constant** is a value that may not be changed during the program’s execution.
- C++ supports two different kinds of constants:
	- **[[Named constants]]** 
	- **[[Literals|Literal constants]]** 


---

### Prefer constant variables to preprocessor macros
- There are (at least) three major problems:
	- The biggest issue is that macros don’t follow normal C++ scoping rules. Once a macro is \#defined, all subsequent occurrences of the macro’s name in the current file will be replaced.
	- Second, it is often harder to debug code using macros. Although your source code will have the macro’s name, the compiler and debugger never see the macro because it has already been replaced before they run. Many debuggers are unable to inspect a macro’s value, and often have limited capabilities when working with macros.
	- Third, macro substitution behaves differently than everything else in C++. Inadvertent mistakes can be easily made as a result.