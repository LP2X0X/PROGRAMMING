---
tags: cpp, term, fundamental
---
### Define - Declare
- When you gave an [[Object|object]] a name and a [[Data type|data type]], you **defined** it.

```cpp
int x; // define an integer variable named x
```

---

### Assignment
- After a variable has been defined, you can give it a value using the = operator. This process is called **assignment**.

```cpp
int x;
x = 5; // assignment of value 5 into variable x
```

---

### Initialization
- When an object is defined, you can optionally give it an initial value. The process of specifying an initial value for an object is called **initialization**, and the syntax used to initialize an object is called an **initializer**.

```cpp
int width {5} // define variable width and initialize with initial value 5
```
- There are different form of initialization:
	- *Default initialization*: When no initializer is provided.
	- *Copy initialization*: When an initial value is provided with an equal sign. Copy initialization is also used whenever values are implicitly copied or converted, such as when passing arguments to a function by value, returning from a function by value, or catching exceptions by value.
	- *Direct initialization*: When an initial value is provided inside parenthesis. One of the reasons direct initialization had fallen out of favor is because it makes it hard to differentiate variables from functions.
	- *List initialization*: The modern way to initialize objects in C++ is to use a form of initialization that makes use of curly braces.
	- *Value initialization* and *Zero initialization*: When a variable is initialized using empty braces. In most cases, **value initialization** will initialize the variable to zero (or empty, if that’s more appropriate for a given type). In such cases where zeroing occurs, this is called **zero initialization**.

```cpp
int a;         // no initializer (default initialization)
int b = 5;     // initial value after equals sign (copy initialization)
int c( 6 );    // initial value in parenthesis (direct initialization)

// List initialization methods (C++11) (preferred)
int d { 7 };   // initial value in braces (direct list initialization)
int e = { 8 }; // initial value in braces after equals sign (copy list initialization)
int f {};      // initializer is empty braces (value initialization)
```
- Benefits of List initialization:
	- The primary benefit of list initialization is that “narrowing conversions” are disallowed. This means that if you try to list initialize a variable using a value that the variable can not safely hold, the compiler is required to produce a diagnostic (usually an error).
	- It supports initialization with lists of values.

```ad-tip
Best practice: Initialize your variables upon creation.
```