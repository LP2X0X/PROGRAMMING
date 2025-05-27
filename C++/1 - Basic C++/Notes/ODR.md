---
tags: cpp, rule
---


- The **One Definition Rule** (or ODR for short) is a well-known rule in C++. The ODR has three parts:
	1. Within a _file_, each function, variable, type, or template in a given scope can only have one definition. Definitions occurring in different scopes (e.g. local variables defined inside different functions, or functions defined inside different namespaces) do not violate this rule.
	2. Within a _program_, each function or variable in a given scope can only have one definition. This rule exists because programs can have more than one file. Functions and variables not visible to the linker are excluded from this rule.
	3. Types, templates, inline functions, and inline variables are allowed to have duplicate definitions in different files, so long as each definition is identical.

```ad-note
Violating part 1 of the ODR will cause the compiler to issue a redefinition error. 
Violating ODR part 2 will cause the linker to issue a redefinition error. 
Violating ODR part 3 will cause undefined behavior.
```