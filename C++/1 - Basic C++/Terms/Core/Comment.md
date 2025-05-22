---
tags: cpp, term, fundamental
---

A **comment** is a programmer-readable note that is inserted directly into the source code of the program. Comments are ignored by the compiler and are for the programmer's use only.

---
### There are two styles of comment:
- The `//` symbol begins a C++ single-line comment, which tells the compiler to ignore everything from the `//` symbol to the end of the line. They are usually place above or to the right of the code.
- The `/*` and `*/` pair of symbols denotes a C-style multi-line comment. Everything in between the symbols is ignored.
---
### Proper use of comments
- To describe **what** the library, function or program does.
- To describe **how** the code is going to accomplish its goal.
- To describe **why** the code is doing something.

```ad-note
Comment your code liberally, and write your comments as if speaking to someone who has no idea what the code does. Don’t assume you’ll remember why you made specific choices.
```

````ad-warning
Don’t use multi-line comments inside other multi-line comments. Wrapping single-line comments inside a multi-line comment is okay.
```cpp
/* This is a multi-line /* comment */ this is not inside the comment */
// The above comment ends at the first */, not the second */
````