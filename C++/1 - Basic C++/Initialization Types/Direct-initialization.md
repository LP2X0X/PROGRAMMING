---
tags: cpp, term, fundamental
---
- Initializes an object from explicit set of constructor arguments.
- The term **direct** is used because:
	- It **directly uses** the constructor of the class with specified arguments.
	- There is **no intermediate step**, like temporary objects or conversions (unless overloaded constructors or implicit conversions are involved).
	- The compiler **chooses the constructor** and **initializes the object in-place**.
- Direct-initialization is more permissive than copy-initialization: copy-initialization only considers non-explicit constructors and non-explicit user-defined conversion functions, while direct-initialization considers all constructors and all user-defined conversion functions. 
- In case of ambiguity between a variable declaration using the direct-initialization syntax (1) (with round parentheses) and a function declaration, the compiler always chooses function declaration.

```ad-tip
Any syntax that directly invokes a constructor without using `=` or an implicit conversion is considered **direct initialization**.
```

```ad-note
Direct-initialization is also used when values are explicitly cast to another type.
```