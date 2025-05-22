---
tags: cpp, term, fundamental
---
- When an initial value is provided inside parenthesis, this is called **direct-initialization**.
- The term **direct** is used because:
	- It **directly uses** the constructor of the class with specified arguments.
	- There is **no intermediate step**, like temporary objects or conversions (unless overloaded constructors or implicit conversions are involved).
	- The compiler **chooses the constructor** and **initializes the object in-place**.

```ad-note
Direct-initialization is also used when values are explicitly cast to another type.
```