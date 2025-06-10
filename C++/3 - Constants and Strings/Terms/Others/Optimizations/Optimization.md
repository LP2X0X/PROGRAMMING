---
tags: cpp, term, fundamental
---

- In programming, **optimization** is the process of modifying software to make it work more efficiently.

---

### Optimization can make programs harder to debug

- When the compiler optimizes a program, the result is that variables, expressions, statements, and function calls may be rearranged, modified, replaced, or removed entirely. Such changes can make it hard to debug a program effectively.
- At runtime, it can be hard to debug compiled code that no longer correlates very well with the original source code.
	- For example, if you try to watch a variable that has been optimized out, the debugger won’t be able to locate the variable. If you try to step into a function that has been optimized away, the debugger will simply skip over it. So if you are debugging your code and the debugger is behaving strangely, this is the most likely reason.
```ad-note
To help minimize such issues, debug builds will typically leave optimizations turned off, so that the compiled code will more closely match the source code.
```