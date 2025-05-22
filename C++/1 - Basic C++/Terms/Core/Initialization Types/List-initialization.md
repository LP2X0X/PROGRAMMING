---
tags: cpp, term, fundamental
---

- The modern way to initialize objects in C++ is to use a form of initialization that makes use of curly braces. This is called **list-initialization** (or **uniform initialization** or **brace initialization**).
- List-initialization comes in two forms:
	```cpp
	int width { 5 };    // direct-list-initialization of initial value 5 into variable width (preferred)
	int height = { 6 }; // copy-list-initialization of initial value 6 into variable height (rarely used)
	```
- Additionally, list-initialization also provides a way to initialize objects with a list of values rather than a single value (which is why it is called “list-initialization”).

---

### List-initialization disallow narrowing conversions
- One of the primary benefits of list-initialization for new C++ programmers is that “narrowing conversions” are disallowed.