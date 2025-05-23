---
tags: cpp, term, fundamental
---

- Initializes an object from a brace-enclosed initializer list. 
- List-initialization comes in two forms:
	```cpp
	int width { 5 };    // direct-list-initialization of initial value 5 into variable width (preferred) // both explicit and non-explicit constructors are considered
	int height = { 6 }; // copy-list-initialization of initial value 6 into variable height (rarely used) // both explicit and non-explicit constructors are considered, but only non-explicit constructors may be called
	```
- Additionally, list-initialization also provides a way to initialize objects with a list of values rather than a single value (which is why it is called “list-initialization”).

---

### List-initialization disallow narrowing conversions
- One of the primary benefits of list-initialization for new C++ programmers is that “narrowing conversions” are disallowed.