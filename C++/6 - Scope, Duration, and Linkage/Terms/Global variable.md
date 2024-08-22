- By convention, global variables are declared at the top of a file, below the includes, in the global namespace.
- Identifiers declared in the global namespace have **global namespace scope** (commonly called **global scope**, and sometimes informally called **file scope**), which means they are visible from the point of declaration until the end of the _file_ in which they are declared.

```ad-note
Variables declared inside a namespace are also global variables.
```

```ad-note
Unlike local variables, which are uninitialized by default, variables with static duration are zero-initialized by default.
```

---

```ad-important
Global variables can be seen and used everywhere in the file. 
Global variables are created when the program starts (before `main()` begins execution), and destroyed when it ends.
```

---

- By convention, some developers prefix global variable identifiers with “g” or “g_” to indicate that they are global. This prefix serves several purposes:
	- It helps avoid naming collisions with other identifiers in the global namespace.
	- It helps prevent inadvertent name shadowing (we discuss this point further in lesson.
	- It helps indicate that the prefixed variables persist beyond the scope of the function, and thus any changes we make to them will also persist.
- Reference: [Hungarian notation](https://en.wikipedia.org/wiki/Hungarian_notation)